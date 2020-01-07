---
title: Developing C Extensions
prev: "/advanced.html"
next: "/advanced/security.html"
---

## Creating Extension Libraries for Ruby[](#creating-extension-libraries-for-ruby)

This document explains how to make extension libraries for Ruby.

### Basic Knowledge[](#basic-knowledge)

In C, variables have types and data do not have types. In contrast, Ruby variables do not have a static type, and data themselves have types, so data will need to be converted between the languages.

Data in Ruby are represented by the C type `VALUE`. Each VALUE data has its data type.

To retrieve C data from a VALUE, you need to:

1.  Identify the VALUE's data type
2.  Convert the VALUE into C data

Converting to the wrong data type may cause serious problems.

#### Data Types[](#data-types)

The Ruby interpreter has the following data types:

* T\_NIL: nil
* T\_OBJECT: ordinary object
* T\_CLASS: class
* T\_MODULE: module
* T\_FLOAT: floating point number
* T\_STRING: string
* T\_REGEXP: regular expression
* T\_ARRAY: array
* T\_HASH: associative array
* T\_STRUCT: (Ruby) structure
* T\_BIGNUM: multi precision integer
* T\_FIXNUM: Fixnum(31bit or 63bit integer)
* T\_COMPLEX: complex number
* T\_RATIONAL: rational number
* T\_FILE: IO
* T\_TRUE: true
* T\_FALSE: false
* T\_DATA: data
* T\_SYMBOL: symbol

In addition, there are several other types used internally:

* T\_ICLASS: included module
* T\_MATCH: MatchData object
* T\_UNDEF: undefined
* T\_NODE: syntax tree node
* T\_ZOMBIE: object awaiting finalization

Most of the types are represented by C structures.

#### Check Data Type of the VALUE[](#check-data-type-of-the-value)

The macro TYPE() defined in ruby.h shows the data type of the VALUE. TYPE() returns the constant number T\_XXXX described above. To handle data types, your code will look something like this:


```
switch (TYPE(obj)) {
  case T_FIXNUM:
    /* process Fixnum */
    break;
  case T_STRING:
    /* process String */
    break;
  case T_ARRAY:
    /* process Array */
    break;
  default:
    /* raise exception */
    rb_raise(rb_eTypeError, "not valid value");
    break;
}
```

There is the data type check function


```
void Check_Type(VALUE value, int type)
```

which raises an exception if the VALUE does not have the type specified.

There are also faster check macros for fixnums and nil.


```ruby
FIXNUM_P(obj)
NIL_P(obj)
```

#### Convert VALUE into C Data[](#convert-value-into-c-data)

The data for type T\_NIL, T\_FALSE, T\_TRUE are nil, false, true respectively. They are singletons for the data type. The equivalent C constants are: Qnil, Qfalse, Qtrue. Note that Qfalse is false in C also (i.e. 0), but not Qnil.

The T\_FIXNUM data is a 31bit or 63bit length fixed integer. This size depends on the size of long: if long is 32bit then T\_FIXNUM is 31bit, if long is 64bit then T\_FIXNUM is 63bit. T\_FIXNUM can be converted to a C integer by using the FIX2INT() macro or FIX2LONG(). Though you have to check that the data is really FIXNUM before using them, they are faster. FIX2LONG() never raises exceptions, but FIX2INT() raises RangeError if the result is bigger or smaller than the size of int. There are also NUM2INT() and NUM2LONG() which converts any Ruby numbers into C integers. These macros include a type check, so an exception will be raised if the conversion failed. NUM2DBL() can be used to retrieve the double float value in the same way.

You can use the macros StringValue() and StringValuePtr() to get a char\* from a VALUE. StringValue(var) replaces var's value with the result of "var.to\_str()". StringValuePtr(var) does the same replacement and returns the char\* representation of var. These macros will skip the replacement if var is a String. Notice that the macros take only the lvalue as their argument, to change the value of var in place.

You can also use the macro named StringValueCStr(). This is just like StringValuePtr(), but always adds a NUL character at the end of the result. If the result contains a NUL character, this macro causes the ArgumentError exception. StringValuePtr() doesn't guarantee the existence of a NUL at the end of the result, and the result may contain NUL.

Other data types have corresponding C structures, e.g. struct RArray for T\_ARRAY etc. The VALUE of the type which has the corresponding structure can be cast to retrieve the pointer to the struct. The casting macro will be of the form RXXXX for each data type; for instance, RARRAY(obj). See "ruby.h". However, we do not recommend to access RXXXX data directly because these data structures are complex. Use corresponding rb\_xxx() functions to access the internal struct. For example, to access an entry of array, use rb\_ary\_entry(ary, offset) and rb\_ary\_store(ary, offset, obj).

There are some accessing macros for structure members, for example `RSTRING_LEN(str)` to get the size of the Ruby String object. The allocated region can be accessed by `RSTRING_PTR(str)`.

Notice: Do not change the value of the structure directly, unless you are responsible for the result. This ends up being the cause of interesting bugs.

#### Convert C Data into VALUE[](#convert-c-data-into-value)

To convert C data to Ruby values:

* FIXNUM: left shift 1 bit, and turn on its least significant bit (LSB).

* Other pointer values: cast to VALUE.

You can determine whether a VALUE is a pointer or not by checking its LSB.

Notice: Ruby does not allow arbitrary pointer values to be a VALUE. They should be pointers to the structures which Ruby knows about. The known structures are defined in <ruby.h>.</ruby.h>

To convert C numbers to Ruby values, use these macros:

* INT2FIX(): for integers within 31bits.
* INT2NUM(): for arbitrary sized integers.

INT2NUM() converts an integer into a Bignum if it is out of the FIXNUM range, but is a bit slower.

#### Manipulating Ruby Data[](#manipulating-ruby-data)

As I already mentioned, it is not recommended to modify an object's internal structure. To manipulate objects, use the functions supplied by the Ruby interpreter. Some (not all) of the useful functions are listed below:

##### String Functions[](#string-functions)

* rb\_str\_new(const char \*ptr, long len): Creates a new Ruby string.

rb\_str\_new2(const char \*ptr)

* rb\_str\_new\_cstr(const char \*ptr): Creates a new Ruby string from a C string. This is equivalent to rb\_str\_new(ptr, strlen(ptr)).

* rb\_str\_new\_literal(const char \*ptr): Creates a new Ruby string from a C string literal.

rb\_sprintf(const char \*format, ...)

* rb\_vsprintf(const char \*format, va\_list ap): Creates a new Ruby string with printf(3) format.
  
  Note: In the format string, "%"PRIsVALUE can be used for Object#to\_s (or Object#inspect if '+' flag is set) output (and related argument must be a VALUE). Since it conflicts with "%i", for integers in format strings, use "%d".

* rb\_str\_append(VALUE str1, VALUE str2): Appends Ruby string str2 to Ruby string str1.

* rb\_str\_cat(VALUE str, const char \*ptr, long len): Appends len bytes of data from ptr to the Ruby string.

rb\_str\_cat2(VALUE str, const char\* ptr)

* rb\_str\_cat\_cstr(VALUE str, const char\* ptr): Appends C string ptr to Ruby string str. This function is equivalent to rb\_str\_cat(str, ptr, strlen(ptr)).

rb\_str\_catf(VALUE str, const char\* format, ...)

* rb\_str\_vcatf(VALUE str, const char\* format, va\_list ap): Appends C string format and successive arguments to Ruby string str according to a printf-like format. These functions are equivalent to rb\_str\_append(str, rb\_sprintf(format, ...)) and rb\_str\_append(str, rb\_vsprintf(format, ap)), respectively.

rb\_enc\_str\_new(const char \*ptr, long len, rb\_encoding \*enc)

* rb\_enc\_str\_new\_cstr(const char \*ptr, rb\_encoding \*enc): Creates a new Ruby string with the specified encoding.

* rb\_enc\_str\_new\_literal(const char \*ptr, rb\_encoding \*enc): Creates a new Ruby string from a C string literal with the specified encoding.

rb\_usascii\_str\_new(const char \*ptr, long len)

* rb\_usascii\_str\_new\_cstr(const char \*ptr): Creates a new Ruby string with encoding US-ASCII.

* rb\_usascii\_str\_new\_literal(const char \*ptr): Creates a new Ruby string from a C string literal with encoding US-ASCII.

rb\_utf8\_str\_new(const char \*ptr, long len)

* rb\_utf8\_str\_new\_cstr(const char \*ptr): Creates a new Ruby string with encoding UTF-8.

* rb\_utf8\_str\_new\_literal(const char \*ptr): Creates a new Ruby string from a C string literal with encoding UTF-8.

* rb\_str\_resize(VALUE str, long len): Resizes a Ruby string to len bytes. If str is not modifiable, this function raises an exception. The length of str must be set in advance. If len is less than the old length the content beyond len bytes is discarded, else if len is greater than the old length the content beyond the old length bytes will not be preserved but will be garbage. Note that RSTRING\_PTR(str) may change by calling this function.

* rb\_str\_set\_len(VALUE str, long len): Sets the length of a Ruby string. If str is not modifiable, this function raises an exception. This function preserves the content up to len bytes, regardless RSTRING\_LEN(str). len must not exceed the capacity of str.

* rb\_str\_modify(VALUE str): Prepares a Ruby string to modify. If str is not modifiable, this function raises an exception, or if the buffer of str is shared, this function allocates new buffer to make it unshared. Always you MUST call this function before modifying the contents using RSTRING\_PTR and/or rb\_str\_set\_len.

##### Array Functions[](#array-functions)

* rb\_ary\_new(): Creates an array with no elements.

rb\_ary\_new2(long len)

* rb\_ary\_new\_capa(long len): Creates an array with no elements, allocating internal buffer for len elements.

rb\_ary\_new3(long n, ...)

* rb\_ary\_new\_from\_args(long n, ...): Creates an n-element array from the arguments.

rb\_ary\_new4(long n, VALUE \*elts)

* rb\_ary\_new\_from\_values(long n, VALUE \*elts): Creates an n-element array from a C array.

* rb\_ary\_to\_ary(VALUE obj): Converts the object into an array. Equivalent to `Object#to_ary`.

There are many functions to operate an array. They may dump core if other types are given.

* rb\_ary\_aref(int argc, const VALUE \*argv, VALUE ary): Equivalent to `Array#[]`.

* rb\_ary\_entry(VALUE ary, long offset): ary\[offset\]

* rb\_ary\_store(VALUE ary, long offset, VALUE obj): ary\[offset\] = obj

* rb\_ary\_subseq(VALUE ary, long beg, long len): ary\[beg, len\]

rb\_ary\_push(VALUE ary, VALUE val) rb\_ary\_pop(VALUE ary) rb\_ary\_shift(VALUE ary)

* rb\_ary\_unshift(VALUE ary, VALUE val): ary.push, ary.pop, ary.shift, ary.unshift

* rb\_ary\_cat(VALUE ary, const VALUE \*ptr, long len): Appends len elements of objects from ptr to the array.

### Extending Ruby with C[](#extending-ruby-with-c)

#### Adding New Features to Ruby[](#adding-new-features-to-ruby)

You can add new features (classes, methods, etc.) to the Ruby interpreter. Ruby provides APIs for defining the following things:

* Classes, Modules
* Methods, Singleton Methods
* Constants

##### Class and Module Definition[](#class-and-module-definition)

To define a class or module, use the functions below:


```
VALUE rb_define_class(const char *name, VALUE super)
VALUE rb_define_module(const char *name)
```

These functions return the newly created class or module. You may want to save this reference into a variable to use later.

To define nested classes or modules, use the functions below:


```
VALUE rb_define_class_under(VALUE outer, const char *name, VALUE super)
VALUE rb_define_module_under(VALUE outer, const char *name)
```

##### Method and Singleton Method Definition[](#method-and-singleton-method-definition)

To define methods or singleton methods, use these functions:


```
void rb_define_method(VALUE klass, const char *name,
                      VALUE (*func)(ANYARGS), int argc)

void rb_define_singleton_method(VALUE object, const char *name,
                                VALUE (*func)(ANYARGS), int argc)
```

The `argc` represents the number of the arguments to the C function, which must be less than 17. But I doubt you'll need that many.

If `argc` is negative, it specifies the calling sequence, not number of the arguments.

If argc is -1, the function will be called as:


```
VALUE func(int argc, VALUE *argv, VALUE obj)
```

where argc is the actual number of arguments, argv is the C array of the arguments, and obj is the receiver.

If argc is -2, the arguments are passed in a Ruby array. The function will be called like:


```
VALUE func(VALUE obj, VALUE args)
```

where obj is the receiver, and args is the Ruby array containing actual arguments.

There are some more functions to define methods. One takes an ID as the name of method to be defined. See also ID or Symbol below.


```
void rb_define_method_id(VALUE klass, ID name,
                         VALUE (*func)(ANYARGS), int argc)
```

There are two functions to define private/protected methods:


```
void rb_define_private_method(VALUE klass, const char *name,
                              VALUE (*func)(ANYARGS), int argc)
void rb_define_protected_method(VALUE klass, const char *name,
                                VALUE (*func)(ANYARGS), int argc)
```

At last, rb\_define\_module\_function defines a module function, which are private AND singleton methods of the module. For example, sqrt is a module function defined in the Math module. It can be called in the following way:


```ruby
Math.sqrt(4)
```

or


```ruby
include Math
sqrt(4)
```

To define module functions, use:


```
void rb_define_module_function(VALUE module, const char *name,
                               VALUE (*func)(ANYARGS), int argc)
```

In addition, function-like methods, which are private methods defined in the Kernel module, can be defined using:


```
void rb_define_global_function(const char *name, VALUE (*func)(ANYARGS), int argc)
```

To define an alias for the method,


```
void rb_define_alias(VALUE module, const char* new, const char* old);
```

To define a reader/writer for an attribute,


```
void rb_define_attr(VALUE klass, const char *name, int read, int write)
```

To define and undefine the `allocate` class method,


```
void rb_define_alloc_func(VALUE klass, VALUE (*func)(VALUE klass));
void rb_undef_alloc_func(VALUE klass);
```

func has to take the klass as the argument and return a newly allocated instance. This instance should be as empty as possible, without any expensive (including external) resources.

If you are overriding an existing method of any ancestor of your class, you may rely on:


```
VALUE rb_call_super(int argc, const VALUE *argv)
```

To specify whether keyword arguments are passed when calling super:


```
VALUE rb_call_super(int argc, const VALUE *argv, int kw_splat)
```

`kw_splat` can have these possible values (used by all methods that accept `kw_splat` argument):

* RB\_NO\_KEYWORDS: Do not pass keywords
* RB\_PASS\_KEYWORDS: Pass keywords, final argument should be a hash of keywords
* RB\_PASS\_EMPTY\_KEYWORDS: Pass empty keywords (not included in arguments) (this will be removed in Ruby 3.0)

* RB\_PASS\_CALLED\_KEYWORDS: Pass keywords if current method was called with keywords, useful for argument delegation

To achieve the receiver of the current scope (if no other way is available), you can use:


```ruby
VALUE rb_current_receiver(void)
```

##### Constant Definition[](#constant-definition)

We have 2 functions to define constants:


```
void rb_define_const(VALUE klass, const char *name, VALUE val)
void rb_define_global_const(const char *name, VALUE val)
```

The former is to define a constant under specified class/module. The latter is to define a global constant.

#### Use Ruby Features from C[](#use-ruby-features-from-c)

There are several ways to invoke Ruby's features from C code.

##### Evaluate Ruby Programs in a String[](#evaluate-ruby-programs-in-a-string)

The easiest way to use Ruby's functionality from a C program is to evaluate the string as Ruby program. This function will do the job:


```ruby
VALUE rb_eval_string(const char *str)
```

Evaluation is done under the current context, thus current local variables of the innermost method (which is defined by Ruby) can be accessed.

Note that the evaluation can raise an exception. There is a safer function:


```
VALUE rb_eval_string_protect(const char *str, int *state)
```

It returns nil when an error occurred. Moreover, \*state is zero if str was successfully evaluated, or nonzero otherwise.

##### ID or Symbol[](#id-or-symbol)

You can invoke methods directly, without parsing the string. First I need to explain about ID. ID is the integer number to represent Ruby's identifiers such as variable names. The Ruby data type corresponding to ID is Symbol. It can be accessed from Ruby in the form:


```ruby
:Identifier
```

or


```ruby
:"any kind of string"
```

You can get the ID value from a string within C code by using


```ruby
rb_intern(const char *name)
rb_intern_str(VALUE name)
```

You can retrieve ID from Ruby object (Symbol or String) given as an argument by using


```
rb_to_id(VALUE symbol)
rb_check_id(volatile VALUE *name)
rb_check_id_cstr(const char *name, long len, rb_encoding *enc)
```

These functions try to convert the argument to a String if it was not a Symbol nor a String. The second function stores the converted result into \*name, and returns 0 if the string is not a known symbol. After this function returned a non-zero value, \*name is always a Symbol or a String, otherwise it is a String if the result is 0. The third function takes NUL-terminated C string, not Ruby VALUE.

You can retrieve Symbol from Ruby object (Symbol or String) given as an argument by using


```
rb_to_symbol(VALUE name)
rb_check_symbol(volatile VALUE *namep)
rb_check_symbol_cstr(const char *ptr, long len, rb_encoding *enc)
```

These functions are similar to above functions except that these return a Symbol instead of an ID.

You can convert C ID to Ruby Symbol by using


```ruby
VALUE ID2SYM(ID id)
```

and to convert Ruby Symbol object to ID, use


```ruby
ID SYM2ID(VALUE symbol)
```

##### Invoke Ruby Method from C[](#invoke-ruby-method-from-c)

To invoke methods directly, you can use the function below


```
VALUE rb_funcall(VALUE recv, ID mid, int argc, ...)
```

This function invokes a method on the recv, with the method name specified by the symbol mid.

##### Accessing the Variables and Constants[](#accessing-the-variables-and-constants)

You can access class variables and instance variables using access functions. Also, global variables can be shared between both environments. There's no way to access Ruby's local variables.

The functions to access/modify instance variables are below:


```
VALUE rb_ivar_get(VALUE obj, ID id)
VALUE rb_ivar_set(VALUE obj, ID id, VALUE val)
```

id must be the symbol, which can be retrieved by rb\_intern().

To access the constants of the class/module:


```
VALUE rb_const_get(VALUE obj, ID id)
```

See also Constant Definition above.

### Information Sharing Between Ruby and C[](#information-sharing-between-ruby-and-c)

#### Ruby Constants That Can Be Accessed From C[](#ruby-constants-that-can-be-accessed-from-c)

As stated in section 1.3, the following Ruby constants can be referred from C.

Qtrue

* Qfalse: Boolean values. Qfalse is false in C also (i.e. 0).

* Qnil: Ruby nil in C scope.

#### Global Variables Shared Between C and Ruby[](#global-variables-shared-between-c-and-ruby)

Information can be shared between the two environments using shared global variables. To define them, you can use functions listed below:


```
void rb_define_variable(const char *name, VALUE *var)
```

This function defines the variable which is shared by both environments. The value of the global variable pointed to by `var` can be accessed through Ruby's global variable named `name`.

You can define read-only (from Ruby, of course) variables using the function below.


```
void rb_define_readonly_variable(const char *name, VALUE *var)
```

You can define hooked variables. The accessor functions (getter and setter) are called on access to the hooked variables.


```
void rb_define_hooked_variable(const char *name, VALUE *var,
                               VALUE (*getter)(), void (*setter)())
```

If you need to supply either setter or getter, just supply 0 for the hook you don't need. If both hooks are 0, rb\_define\_hooked\_variable() works just like rb\_define\_variable().

The prototypes of the getter and setter functions are as follows:


```
VALUE (*getter)(ID id, VALUE *var);
void (*setter)(VALUE val, ID id, VALUE *var);
```

Also you can define a Ruby global variable without a corresponding C variable. The value of the variable will be set/get only by hooks.


```
void rb_define_virtual_variable(const char *name,
                                VALUE (*getter)(), void (*setter)())
```

The prototypes of the getter and setter functions are as follows:


```
VALUE (*getter)(ID id);
void (*setter)(VALUE val, ID id);
```

#### Encapsulate C Data into a Ruby Object[](#encapsulate-c-data-into-a-ruby-object)

Sometimes you need to expose your struct in the C world as a Ruby object. In a situation like this, making use of the TypedData\_XXX macro family, the pointer to the struct and the Ruby object can be mutually converted.

-- The old (non-Typed) Data\_XXX macro family has been deprecated. In the future version of Ruby, it is possible old macros will not work. ++

##### C struct to Ruby object[](#c-struct-to-ruby-object)

You can convert sval, a pointer to your struct, into a Ruby object with the next macro.


```ruby
TypedData_Wrap_Struct(klass, data_type, sval)
```

TypedData\_Wrap\_Struct() returns a created Ruby object as a VALUE.

The klass argument is the class for the object. data\_type is a pointer to a const rb\_data\_type\_t which describes how Ruby should manage the struct.

It is recommended that klass derives from a special class called Data (rb\_cData) but not from Object or other ordinal classes. If it doesn't, you have to call rb\_undef\_alloc\_func(klass).

rb\_data\_type\_t is defined like this. Let's take a look at each member of the struct.


```
typedef struct rb_data_type_struct rb_data_type_t;

struct rb_data_type_struct {
        const char *wrap_struct_name;
        struct {
                void (*dmark)(void*);
                void (*dfree)(void*);
                size_t (*dsize)(const void *);
                void *reserved[2];
        } function;
        const rb_data_type_t *parent;
        void *data;
        VALUE flags;
};
```

wrap\_struct\_name is an identifier of this instance of the struct. It is basically used for collecting and emitting statistics. So the identifier must be unique in the process, but doesn't need to be valid as a C or Ruby identifier.

These dmark / dfree functions are invoked during GC execution. No object allocations are allowed during it, so do not allocate ruby objects inside them.

dmark is a function to mark Ruby objects referred from your struct. It must mark all references from your struct with rb\_gc\_mark or its family if your struct keeps such references.

-- Note that it is recommended to avoid such a reference. ++

dfree is a function to free the pointer allocation. If this is -1, the pointer will be just freed.

dsize calculates memory consumption in bytes by the struct. Its parameter is a pointer to your struct. You can pass 0 as dsize if it is hard to implement such a function. But it is still recommended to avoid 0.

You have to fill reserved and parent with 0.

You can fill "data" with an arbitrary value for your use. Ruby does nothing with the member.

flags is a bitwise-OR of the following flag values. Since they require deep understanding of garbage collector in Ruby, you can just set 0 to flags if you are not sure.

* RUBY\_TYPED\_FREE\_IMMEDIATELY: This flag makes the garbage collector immediately invoke dfree() during GC when it need to free your struct. You can specify this flag if the dfree never unlocks Ruby's internal lock (GVL).
  
  If this flag is not set, Ruby defers invocation of dfree() and invokes dfree() at the same time as finalizers.

* RUBY\_TYPED\_WB\_PROTECTED: It shows that implementation of the object supports write barriers. If this flag is set, Ruby is better able to do garbage collection of the object.
  
  When it is set, however, you are responsible for putting write barriers in all implementations of methods of that object as appropriate. Otherwise Ruby might crash while running.
  
  More about write barriers can be found in "Generational GC" in Appendix D.

You can allocate and wrap the structure in one step.


```ruby
TypedData_Make_Struct(klass, type, data_type, sval)
```

This macro returns an allocated Data object, wrapping the pointer to the structure, which is also allocated. This macro works like:


```ruby
(sval = ZALLOC(type), TypedData_Wrap_Struct(klass, data_type, sval))
```

Arguments klass and data\_type work like their counterparts in TypedData\_Wrap\_Struct(). A pointer to the allocated structure will be assigned to sval, which should be a pointer of the type specified.

##### Ruby object to C struct[](#ruby-object-to-c-struct)

To retrieve the C pointer from the Data object, use the macro TypedData\_Get\_Struct().


```
TypedData_Get_Struct(obj, type, &data_type, sval)
```

A pointer to the structure will be assigned to the variable sval.

See the example below for details.

### Example - Creating the dbm Extension[](#example---creating-the-dbm-extension)

OK, here's the example of making an extension library. This is the extension to access DBMs. The full source is included in the ext/ directory in the Ruby's source tree.

#### Make the Directory[](#make-the-directory)


```
% mkdir ext/dbm
```

Make a directory for the extension library under ext directory.

#### Design the Library[](#design-the-library)

You need to design the library features, before making it.

#### Write the C Code[](#write-the-c-code)

You need to write C code for your extension library. If your library has only one source file, choosing `` LIBRARY.c`' as a file name is preferred.  On the
other hand, in case your library has multiple source files, avoid choosing
 ``LIBRARY.c`' for a file name.  It may conflict with an intermediate file
``LIBRARY.o`' on some platforms. Note that some functions in mkmf library described below generate a file `` conftest.c`' for checking with compilation. 
You shouldn't choose ``conftest.c\`' as a name of a source file.

Ruby will execute the initializing function named `` Init_LIBRARY`' in the
library.  For example, ``Init\_dbm()\`' will be executed when loading the library.

Here's the example of an initializing function.


```
void
Init_dbm(void)
{
    /* define DBM class */
    VALUE cDBM = rb_define_class("DBM", rb_cObject);
    /* DBM includes Enumerable module */
    rb_include_module(cDBM, rb_mEnumerable);

    /* DBM has class method open(): arguments are received as C array */
    rb_define_singleton_method(cDBM, "open", fdbm_s_open, -1);

    /* DBM instance method close(): no args */
    rb_define_method(cDBM, "close", fdbm_close, 0);
    /* DBM instance method []: 1 argument */
    rb_define_method(cDBM, "[]", fdbm_fetch, 1);

    /* ... */

    /* ID for a instance variable to store DBM data */
    id_dbm = rb_intern("dbm");
}
```

The dbm extension wraps the dbm struct in the C environment using TypedData\_Make\_Struct.


```
struct dbmdata {
    int  di_size;
    DBM *di_dbm;
};

static const rb_data_type_t dbm_type = {
    "dbm",
    {0, free_dbm, memsize_dbm,},
    0, 0,
    RUBY_TYPED_FREE_IMMEDIATELY,
};

obj = TypedData_Make_Struct(klass, struct dbmdata, &dbm_type, dbmp);
```

This code wraps the dbmdata structure into a Ruby object. We avoid wrapping DBM\* directly, because we want to cache size information.

To retrieve the dbmdata structure from a Ruby object, we define the following macro:


```
\#define GetDBM(obj, dbmp) do {\
    TypedData_Get_Struct((obj), struct dbmdata, &dbm_type, (dbmp));\
    if ((dbmp) == 0) closed_dbm();\
    if ((dbmp)->di_dbm == 0) closed_dbm();\
} while (0)
```

This sort of complicated macro does the retrieving and close checking for the DBM.

There are three kinds of way to receive method arguments. First, methods with a fixed number of arguments receive arguments like this:


```
static VALUE
fdbm_delete(VALUE obj, VALUE keystr)
{
      /* ... */
}
```

The first argument of the C function is the self, the rest are the arguments to the method.

Second, methods with an arbitrary number of arguments receive arguments like this:


```
static VALUE
fdbm_s_open(int argc, VALUE *argv, VALUE klass)
{
    /* ... */
    if (rb_scan_args(argc, argv, "11", &file, &vmode) == 1) {
        mode = 0666;          /* default value */
    }
    /* ... */
}
```

The first argument is the number of method arguments, the second argument is the C array of the method arguments, and the third argument is the receiver of the method.

You can use the function rb\_scan\_args() to check and retrieve the arguments. The third argument is a string that specifies how to capture method arguments and assign them to the following VALUE references.

You can just check the argument number with rb\_check\_arity(), this is handy in the case you want to treat the arguments as a list.

The following is an example of a method that takes arguments by Ruby's array:


```
static VALUE
thread_initialize(VALUE thread, VALUE args)
{
    /* ... */
}
```

The first argument is the receiver, the second one is the Ruby array which contains the arguments to the method.

**Notice**\: GC should know about global variables which refer to Ruby's objects, but are not exported to the Ruby world. You need to protect them by


```ruby
void rb_global_variable(VALUE *var)
```

or the objects themselves by


```ruby
void rb_gc_register_mark_object(VALUE object)
```

#### Prepare extconf.rb[](#prepare-extconfrb)

If the file named extconf.rb exists, it will be executed to generate Makefile.

extconf.rb is the file for checking compilation conditions etc. You need to put


```ruby
require 'mkmf'
```

at the top of the file. You can use the functions below to check various conditions.


```
have_macro(macro[, headers[, opt]]): check whether macro is defined
have_library(lib[, func[, headers[, opt]]]): check whether library containing function exists
find_library(lib[, func, *paths]): find library from paths
have_func(func[, headers[, opt]): check whether function exists
have_var(var[, headers[, opt]]): check whether variable exists
have_header(header[, preheaders[, opt]]): check whether header file exists
find_header(header, *paths): find header from paths
have_framework(fw): check whether framework exists (for MacOS X)
have_struct_member(type, member[, headers[, opt]]): check whether struct has member
have_type(type[, headers[, opt]]): check whether type exists
find_type(type, opt, *headers): check whether type exists in headers
have_const(const[, headers[, opt]]): check whether constant is defined
check_sizeof(type[, headers[, opts]]): check size of type
check_signedness(type[, headers[, opts]]): check signedness of type
convertible_int(type[, headers[, opts]]): find convertible integer type
find_executable(bin[, path]): find executable file path
create_header(header): generate configured header
create_makefile(target[, target_prefix]): generate Makefile
```

See MakeMakefile for full documentation of these functions.

The value of the variables below will affect the Makefile.


```
$CFLAGS: included in CFLAGS make variable (such as -O)
$CPPFLAGS: included in CPPFLAGS make variable (such as -I, -D)
$LDFLAGS: included in LDFLAGS make variable (such as -L)
$objs: list of object file names
```

Normally, the object files list is automatically generated by searching source files, but you must define them explicitly if any sources will be generated while building.

If a compilation condition is not fulfilled, you should not call \`\`create\_makefile\`'. The Makefile will not be generated, compilation will not be done.

#### Prepare Depend (Optional)[](#prepare-depend-optional)

If the file named depend exists, Makefile will include that file to check dependencies. You can make this file by invoking


```
% gcc -MM *.c > depend
```

It's harmless. Prepare it.

#### Generate Makefile[](#generate-makefile)

Try generating the Makefile by:


```ruby
ruby extconf.rb
```

If the library should be installed under vendor\_ruby directory instead of site\_ruby directory, use --vendor option as follows.


```ruby
ruby extconf.rb --vendor
```

You don't need this step if you put the extension library under the ext directory of the ruby source tree. In that case, compilation of the interpreter will do this step for you.

#### Run make[](#run-make)

Type


```ruby
make
```

to compile your extension. You don't need this step either if you have put the extension library under the ext directory of the ruby source tree.

#### Debug[](#debug)

You may need to rb\_debug the extension. Extensions can be linked statically by adding the directory name in the ext/Setup file so that you can inspect the extension with the debugger.

#### Done! Now You Have the Extension Library[](#done-now-you-have-the-extension-library)

You can do anything you want with your library. The author of Ruby will not claim any restrictions on your code depending on the Ruby API. Feel free to use, modify, distribute or sell your program.

### Appendix A. Ruby Source Files Overview[](#appendix-a-ruby-source-files-overview)

#### Ruby Language Core[](#ruby-language-core)

* class.c: classes and modules
* error.c: exception classes and exception mechanism
* gc.c: memory management
* load.c: library loading
* object.c: objects
* variable.c: variables and constants

#### Ruby Syntax Parser[](#ruby-syntax-parser)

* parse.y: grammar definition
* parse.c: automatically generated from parse.y
* defs/keywords: reserved keywords
* lex.c: automatically generated from keywords

#### Ruby Evaluator (a.k.a. YARV)[](#ruby-evaluator-aka-yarv)


```
compile.c
eval.c
eval_error.c
eval_jump.c
eval_safe.c
insns.def           : definition of VM instructions
iseq.c              : implementation of VM::ISeq
thread.c            : thread management and context switching
thread_win32.c      : thread implementation
thread_pthread.c    : ditto
vm.c
vm_dump.c
vm_eval.c
vm_exec.c
vm_insnhelper.c
vm_method.c

defs/opt_insns_unif.def  : instruction unification
defs/opt_operand.def     : definitions for optimization

  -> insn*.inc           : automatically generated
  -> opt*.inc            : automatically generated
  -> vm.inc              : automatically generated
```

#### Regular Expression Engine (Oniguruma)[](#regular-expression-engine-oniguruma)


```ruby
regex.c
regcomp.c
regenc.c
regerror.c
regexec.c
regparse.c
regsyntax.c
```

#### Utility Functions[](#utility-functions)

* debug.c: debug symbols for C debugger
* dln.c: dynamic loading
* st.c: general purpose hash table
* strftime.c: formatting times
* util.c: misc utilities

#### Ruby Interpreter Implementation[](#ruby-interpreter-implementation)


```ruby
dmyext.c
dmydln.c
dmyencoding.c
id.c
inits.c
main.c
ruby.c
version.c

gem_prelude.rb
prelude.rb
```

#### Class Library[](#class-library)

* array.c: Array
* bignum.c: Bignum
* compar.c: Comparable
* complex.c: Complex
* cont.c: Fiber, Continuation
* dir.c: Dir
* enum.c: Enumerable
* enumerator.c: Enumerator
* file.c: File
* hash.c: Hash
* io.c: IO
* marshal.c: Marshal
* math.c: Math
* numeric.c: Numeric, Integer, Fixnum, Float
* pack.c: `Array#pack`, `String#unpack`
* proc.c: Binding, Proc
* process.c: Process
* random.c: random number
* range.c: Range
* rational.c: Rational
* re.c: Regexp, MatchData
* signal.c: Signal
* sprintf.c: `String#sprintf`
* string.c: String
* struct.c: Struct
* time.c: Time

* defs/known\_errors.def: Errno::\* exception classes
* -> known\_errors.inc: automatically generated

#### Multilingualization[](#multilingualization)

* encoding.c: Encoding
* transcode.c: Encoding::Converter
* enc/\*.c: encoding classes
* enc/trans/\*: codepoint mapping tables

#### goruby Interpreter Implementation[](#goruby-interpreter-implementation)


```
goruby.c
golf_prelude.rb     : goruby specific libraries.
  -> golf_prelude.c : automatically generated
```

### Appendix B. Ruby Extension API Reference[](#appendix-b-ruby-extension-api-reference)

#### Types[](#types)

* VALUE: The type for the Ruby object. Actual structures are defined in ruby.h, such as struct RString, etc. To refer the values in structures, use casting macros like RSTRING(obj).

#### Variables and Constants[](#variables-and-constants)

* Qnil: nil object

* Qtrue: true object (default true value)

* Qfalse: false object

#### C Pointer Wrapping[](#c-pointer-wrapping)

* Data\_Wrap\_Struct(VALUE klass, void (*mark)(), void (*free)(), void \*sval): Wrap a C pointer into a Ruby object. If object has references to other Ruby objects, they should be marked by using the mark function during the GC process. Otherwise, mark should be 0. When this object is no longer referred by anywhere, the pointer will be discarded by free function.

* Data\_Make\_Struct(klass, type, mark, free, sval): This macro allocates memory using malloc(), assigns it to the variable sval, and returns the DATA encapsulating the pointer to memory region.

* Data\_Get\_Struct(data, type, sval): This macro retrieves the pointer value from DATA, and assigns it to the variable sval.

#### Checking Data Types[](#checking-data-types)

* RB\_TYPE\_P(value, type): Is `value` an internal type (T\_NIL, T\_FIXNUM, etc.)?

* TYPE(value): Internal type (T\_NIL, T\_FIXNUM, etc.)

* FIXNUM\_P(value): Is `value` a Fixnum?

* NIL\_P(value): Is `value` nil?

* RB\_INTEGER\_TYPE\_P(value): Is `value` an Integer?

* RB\_FLOAT\_TYPE\_P(value): Is `value` a Float?

* void Check\_Type(VALUE value, int type): Ensures `value` is of the given internal `type` or raises a TypeError

#### Data Type Conversion[](#data-type-conversion)

* FIX2INT(value), INT2FIX(i): Fixnum <-> integer

* FIX2LONG(value), LONG2FIX(l): Fixnum <-> long

* NUM2INT(value), INT2NUM(i): Numeric <-> integer

* NUM2UINT(value), UINT2NUM(ui): Numeric <-> unsigned integer

* NUM2LONG(value), LONG2NUM(l): Numeric <-> long

* NUM2ULONG(value), ULONG2NUM(ul): Numeric <-> unsigned long

* NUM2LL(value), LL2NUM(ll): Numeric <-> long long

* NUM2ULL(value), ULL2NUM(ull): Numeric <-> unsigned long long

* NUM2OFFT(value), OFFT2NUM(off): Numeric <-> off\_t

* NUM2SIZET(value), SIZET2NUM(size): Numeric <-> size\_t

* NUM2SSIZET(value), SSIZET2NUM(ssize): Numeric <-> ssize\_t

* rb\_integer\_pack(value, words, numwords, wordsize, nails, flags), rb\_integer\_unpack(words, numwords, wordsize, nails, flags): Numeric <-> Arbitrary size integer buffer

* NUM2DBL(value): Numeric -> double

* rb\_float\_new(f): double -> Float

* RSTRING\_LEN(str): String -> length of String data in bytes

* RSTRING\_PTR(str): String -> pointer to String data Note that the result pointer may not be NUL-terminated

* StringValue(value): Object with `#to_str` -> String

* StringValuePtr(value): Object with `#to_str` -> pointer to String data

* StringValueCStr(value): Object with `#to_str` -> pointer to String data without NUL bytes It is guaranteed that the result data is NUL-terminated

* rb\_str\_new2(s): char \* -> String

#### Defining Classes and Modules[](#defining-classes-and-modules)

* VALUE rb\_define\_class(const char \*name, VALUE super): Defines a new Ruby class as a subclass of super.

* VALUE rb\_define\_class\_under(VALUE module, const char \*name, VALUE super): Creates a new Ruby class as a subclass of super, under the module's namespace.

* VALUE rb\_define\_module(const char \*name): Defines a new Ruby module.

* VALUE rb\_define\_module\_under(VALUE module, const char \*name): Defines a new Ruby module under the module's namespace.

* void rb\_include\_module(VALUE klass, VALUE module): Includes module into class. If class already includes it, just ignored.

* void rb\_extend\_object(VALUE object, VALUE module): Extend the object with the module's attributes.

#### Defining Global Variables[](#defining-global-variables)

* void rb\_define\_variable(const char \*name, VALUE \*var): Defines a global variable which is shared between C and Ruby. If name contains a character which is not allowed to be part of the symbol, it can't be seen from Ruby programs.

* void rb\_define\_readonly\_variable(const char \*name, VALUE \*var): Defines a read-only global variable. Works just like rb\_define\_variable(), except the defined variable is read-only.

* void rb\_define\_virtual\_variable(const char *name, VALUE (*getter)(), void (\*setter)()): Defines a virtual variable, whose behavior is defined by a pair of C functions. The getter function is called when the variable is referenced. The setter function is called when the variable is set to a value. The prototype for getter/setter functions are:
  
  
  ```
    VALUE getter(ID id)
    void setter(VALUE val, ID id)
  ```
  
  The getter function must return the value for the access.

* void rb\_define\_hooked\_variable(const char *name, VALUE \*var, VALUE (*getter)(), void (\*setter)()): Defines hooked variable. It's a virtual variable with a C variable. The getter is called as
  
  
  ```
    VALUE getter(ID id, VALUE *var)
  ```
  
  returning a new value. The setter is called as
  
  
  ```
    void setter(VALUE val, ID id, VALUE *var)
  ```

* void rb\_global\_variable(VALUE \*var): Tells GC to protect C global variable, which holds Ruby value to be marked.

* void rb\_gc\_register\_mark\_object(VALUE object): Tells GC to protect the `object`, which may not be referenced anywhere.

#### Constant Definition[](#constant-definition-1)

* void rb\_define\_const(VALUE klass, const char \*name, VALUE val): Defines a new constant under the class/module.

* void rb\_define\_global\_const(const char \*name, VALUE val): Defines a global constant. This is just the same as
  
  
  ```ruby
    rb_define_const(rb_cObject, name, val)
  ```

#### Method Definition[](#method-definition)

* rb\_define\_method(VALUE klass, const char *name, VALUE (*func)(ANYARGS), int argc): Defines a method for the class. func is the function pointer. argc is the number of arguments. if argc is -1, the function will receive 3 arguments: argc, argv, and self. if argc is -2, the function will receive 2 arguments, self and args, where args is a Ruby array of the method arguments.

* rb\_define\_private\_method(VALUE klass, const char *name, VALUE (*func)(ANYARGS), int argc): Defines a private method for the class. Arguments are same as rb\_define\_method().

* rb\_define\_singleton\_method(VALUE klass, const char *name, VALUE (*func)(ANYARGS), int argc): Defines a singleton method. Arguments are same as rb\_define\_method().

* rb\_check\_arity(int argc, int min, int max): Check the number of arguments, argc is in the range of min..max. If max is UNLIMITED\_ARGUMENTS, upper bound is not checked. If argc is out of bounds, an ArgumentError will be raised.

* rb\_scan\_args(int argc, VALUE \*argv, const char \*fmt, ...): Retrieve argument from argc and argv to given VALUE references according to the format string. The format can be described in ABNF as follows:
  
  
  ```
    scan-arg-spec  := param-arg-spec [keyword-arg-spec] [block-arg-spec]
  
    param-arg-spec := pre-arg-spec [post-arg-spec] / post-arg-spec /
                      pre-opt-post-arg-spec
    pre-arg-spec   := num-of-leading-mandatory-args [num-of-optional-args]
    post-arg-spec  := sym-for-variable-length-args
                      [num-of-trailing-mandatory-args]
    pre-opt-post-arg-spec := num-of-leading-mandatory-args num-of-optional-args
                             num-of-trailing-mandatory-args
    keyword-arg-spec := sym-for-keyword-arg
    block-arg-spec := sym-for-block-arg
  
    num-of-leading-mandatory-args  := DIGIT ; The number of leading
                                            ; mandatory arguments
    num-of-optional-args           := DIGIT ; The number of optional
                                            ; arguments
    sym-for-variable-length-args   := "*"   ; Indicates that variable
                                            ; length arguments are
                                            ; captured as a ruby array
    num-of-trailing-mandatory-args := DIGIT ; The number of trailing
                                            ; mandatory arguments
    sym-for-keyword-arg            := ":"   ; Indicates that keyword
                                            ; argument captured as a hash.
                                            ; If keyword arguments are not
                                            ; provided, returns nil.
                                            ;
                                            ; Currently, will also consider
                                            ; final argument as keywords if
                                            ; it is a hash or can be
                                            ; converted to a hash with
                                            ; #to_hash.  When the last
                                            ; argument is nil, it is
                                            ; captured if it is not
                                            ; ambiguous to take it as
                                            ; empty option hash; i.e. '*'
                                            ; is not specified and
                                            ; arguments are given more
                                            ; than sufficient.
                                            ;
                                            ; However, handling final
                                            ; argument as keywords if
                                            ; method was not called with
                                            ; keywords (whether final
                                            ; argument is hash or nil) is
                                            ; deprecated. In that case, a
                                            ; warning will be emitted, and
                                            ; in Ruby 3.0 it will be an error.
    sym-for-block-arg              := "&"   ; Indicates that an iterator
                                            ; block should be captured if
                                            ; given
  ```
  
  For example, "12" means that the method requires at least one argument, and at most receives three (1+2) arguments. So, the format string must be followed by three variable references, which are to be assigned to captured arguments. For omitted arguments, variables are set to Qnil. NULL can be put in place of a variable reference, which means the corresponding captured argument(s) should be just dropped.
  
  The number of given arguments, excluding an option hash or iterator block, is returned.

* rb\_scan\_args\_kw(int kw\_splat, int argc, VALUE \*argv, const char \*fmt, ...): The same as `rb_scan_args`, except the `kw_splat` argument specifies whether keyword arguments are provided (instead of being determined by the call from Ruby to the C function). `kw_splat` should be one of the following values:
  
  RB\_SCAN\_ARGS\_PASS\_CALLED\_KEYWORDS
  : Same behavior as `rb_scan_args`. RB\_SCAN\_ARGS\_KEYWORDS
  : The final argument should be a hash treated as keywords. RB\_SCAN\_ARGS\_EMPTY\_KEYWORDS
  : Don't treat a final hash as keywords. (this will be removed in Ruby 3.0) RB\_SCAN\_ARGS\_LAST\_HASH\_KEYWORDS
  : Treat a final argument as keywords if it is a hash, and not as keywords otherwise.

* int rb\_get\_kwargs(VALUE keyword\_hash, const ID \*table, int required, int optional, VALUE \*values): Retrieves argument VALUEs bound to keywords, which directed by `table` into `values`, deleting retrieved entries from `keyword_hash` along the way. First `required` number of IDs referred by `table` are mandatory, and succeeding `optional` (- `optional` - 1 if `optional` is negative) number of IDs are optional. If a mandatory key is not contained in `keyword_hash`, raises "missing keyword" `ArgumentError`. If an optional key is not present in `keyword_hash`, the corresponding element in `values` is set to `Qundef`. If `optional` is negative, rest of `keyword_hash` are ignored, otherwise raises "unknown keyword" `ArgumentError`.
  
  Be warned, handling keyword arguments in the C API is less efficient than handling them in Ruby. Consider using a Ruby wrapper method around a non-keyword C function. ref: https://bugs.ruby-lang.org/issues/11339

* VALUE rb\_extract\_keywords(VALUE \*original\_hash): Extracts pairs whose key is a symbol into a new hash from a hash object referred by `original_hash`. If the original hash contains non-symbol keys, then they are copied to another hash and the new hash is stored through `original_hash`, else 0 is stored.

#### Invoking Ruby method[](#invoking-ruby-method)

* VALUE rb\_funcall(VALUE recv, ID mid, int narg, ...): Invokes a method. To retrieve mid from a method name, use rb\_intern(). Able to call even private/protected methods.

VALUE rb\_funcall2(VALUE recv, ID mid, int argc, VALUE \*argv)

* VALUE rb\_funcallv(VALUE recv, ID mid, int argc, VALUE \*argv): Invokes a method, passing arguments as an array of values. Able to call even private/protected methods.

* VALUE rb\_funcallv\_kw(VALUE recv, ID mid, int argc, VALUE \*argv, int kw\_splat): Same as rb\_funcallv, using `kw_splat` to determine whether keyword arguments are passed.

* VALUE rb\_funcallv\_public(VALUE recv, ID mid, int argc, VALUE \*argv): Invokes a method, passing arguments as an array of values. Able to call only public methods.

* VALUE rb\_funcallv\_public\_kw(VALUE recv, ID mid, int argc, VALUE \*argv, int kw\_splat): Same as rb\_funcallv\_public, using `kw_splat` to determine whether keyword arguments are passed.

* VALUE rb\_funcall\_passing\_block(VALUE recv, ID mid, int argc, const VALUE\* argv): Same as rb\_funcallv\_public, except is passes the currently active block as the block when calling the method.

* VALUE rb\_funcall\_passing\_block\_kw(VALUE recv, ID mid, int argc, const VALUE\* argv, int kw\_splat): Same as rb\_funcall\_passing\_block, using `kw_splat` to determine whether keyword arguments are passed.

* VALUE rb\_funcall\_with\_block(VALUE recv, ID mid, int argc, const VALUE \*argv, VALUE passed\_procval): Same as rb\_funcallv\_public, except `passed_procval` specifies the block to pass to the method.

* VALUE rb\_funcall\_with\_block\_kw(VALUE recv, ID mid, int argc, const VALUE \*argv, VALUE passed\_procval, int kw\_splat): Same as rb\_funcall\_with\_block, using `kw_splat` to determine whether keyword arguments are passed.

* VALUE rb\_eval\_string(const char \*str): Compiles and executes the string as a Ruby program.

* ID rb\_intern(const char \*name): Returns ID corresponding to the name.

* char \*rb\_id2name(ID id): Returns the name corresponding ID.

* char \*rb\_class2name(VALUE klass): Returns the name of the class.

* int rb\_respond\_to(VALUE obj, ID id): Returns true if the object responds to the message specified by id.

#### Instance Variables[](#instance-variables)

* VALUE rb\_iv\_get(VALUE obj, const char \*name): Retrieve the value of the instance variable. If the name is not prefixed by \`@', that variable shall be inaccessible from Ruby.

* VALUE rb\_iv\_set(VALUE obj, const char \*name, VALUE val): Sets the value of the instance variable.

#### Control Structure[](#control-structure)

* VALUE rb\_block\_call(VALUE recv, ID mid, int argc, VALUE \* argv, VALUE (\*func) (ANYARGS), VALUE data2): Calls a method on the recv, with the method name specified by the symbol mid, with argc arguments in argv, supplying func as the block. When func is called as the block, it will receive the value from yield as the first argument, and data2 as the second argument. When yielded with multiple values (in C, rb\_yield\_values(), rb\_yield\_values2() and rb\_yield\_splat()), data2 is packed as an Array, whereas yielded values can be gotten via argc/argv of the third/fourth arguments.

* VALUE rb\_block\_call\_kw(VALUE recv, ID mid, int argc, VALUE \* argv, VALUE (\*func) (ANYARGS), VALUE data2, int kw\_splat): Same as rb\_funcall\_with\_block, using `kw_splat` to determine whether keyword arguments are passed.

* \[OBSOLETE\] VALUE rb\_iterate(VALUE (*func1)(), VALUE arg1, VALUE (*func2)(), VALUE arg2): Calls the function func1, supplying func2 as the block. func1 will be called with the argument arg1. func2 receives the value from yield as the first argument, arg2 as the second argument.
  
  When rb\_iterate is used in 1.9, func1 has to call some Ruby-level method. This function is obsolete since 1.9; use rb\_block\_call instead.

* VALUE rb\_yield(VALUE val): Yields val as a single argument to the block.

* VALUE rb\_yield\_values(int n, ...): Yields `n` number of arguments to the block, using one C argument per Ruby argument.

* VALUE rb\_yield\_values2(int n, VALUE \*argv): Yields `n` number of arguments to the block, with all Ruby arguments in the C argv array.

* VALUE rb\_yield\_values\_kw(int n, VALUE \*argv, int kw\_splat): Same as rb\_yield\_values2, using `kw_splat` to determine whether keyword arguments are passed.

* VALUE rb\_yield\_splat(VALUE args): Same as rb\_yield\_values2, except arguments are specified by the Ruby array `args`.

* VALUE rb\_yield\_splat\_kw(VALUE args, int kw\_splat): Same as rb\_yield\_splat, using `kw_splat` to determine whether keyword arguments are passed.

* VALUE rb\_rescue(VALUE (*func1)(ANYARGS), VALUE arg1, VALUE (*func2)(ANYARGS), VALUE arg2): Calls the function func1, with arg1 as the argument. If an exception occurs during func1, it calls func2 with arg2 as the first argument and the exception object as the second argument. The return value of rb\_rescue() is the return value from func1 if no exception occurs, from func2 otherwise.

* VALUE rb\_ensure(VALUE (*func1)(ANYARGS), VALUE arg1, VALUE (*func2)(ANYARGS), VALUE arg2): Calls the function func1 with arg1 as the argument, then calls func2 with arg2 if execution terminated. The return value from rb\_ensure() is that of func1 when no exception occurred.

* VALUE rb\_protect(VALUE (\*func) (VALUE), VALUE arg, int \*state): Calls the function func with arg as the argument. If no exception occurred during func, it returns the result of func and \*state is zero. Otherwise, it returns Qnil and sets \*state to nonzero. If state is NULL, it is not set in both cases. You have to clear the error info with rb\_set\_errinfo(Qnil) when ignoring the caught exception.

* void rb\_jump\_tag(int state): Continues the exception caught by rb\_protect() and rb\_eval\_string\_protect(). state must be the returned value from those functions. This function never return to the caller.

* void rb\_iter\_break(): Exits from the current innermost block. This function never return to the caller.

* void rb\_iter\_break\_value(VALUE value): Exits from the current innermost block with the value. The block will return the given argument value. This function never return to the caller.

#### Exceptions and Errors[](#exceptions-and-errors)

* void rb\_warn(const char \*fmt, ...): Prints a warning message according to a printf-like format.

* void rb\_warning(const char \*fmt, ...): Prints a warning message according to a printf-like format, if $VERBOSE is true.

* void rb\_raise(rb\_eRuntimeError, const char \*fmt, ...): Raises RuntimeError. The fmt is a format string just like printf().

* void rb\_raise(VALUE exception, const char \*fmt, ...): Raises a class exception. The fmt is a format string just like printf().

* void rb\_fatal(const char \*fmt, ...): Raises a fatal error, terminates the interpreter. No exception handling will be done for fatal errors, but ensure blocks will be executed.

* void rb\_bug(const char \*fmt, ...): Terminates the interpreter immediately. This function should be called under the situation caused by the bug in the interpreter. No exception handling nor ensure execution will be done.

Note: In the format string, "%"PRIsVALUE can be used for `Object#to_s` (or Object#inspect if '+' flag is set) output (and related argument must be a VALUE). Since it conflicts with "%i", for integers in format strings, use "%d".

#### Threading[](#threading)

As of Ruby 1.9, Ruby supports native 1:1 threading with one kernel thread per Ruby Thread object. Currently, there is a GVL (Global VM Lock) which prevents simultaneous execution of Ruby code which may be released by the rb\_thread\_call\_without\_gvl and rb\_thread\_call\_without\_gvl2 functions. These functions are tricky-to-use and documented in thread.c; do not use them before reading comments in thread.c.

* void rb\_thread\_schedule(void): Give the scheduler a hint to pass execution to another thread.

#### Input/Output (IO) on a single file descriptor[](#inputoutput-io-on-a-single-file-descriptor)

* int rb\_io\_wait\_readable(int fd): Wait indefinitely for the given FD to become readable, allowing other threads to be scheduled. Returns a true value if a read may be performed, false if there is an unrecoverable error.

* int rb\_io\_wait\_writable(int fd): Like rb\_io\_wait\_readable, but for writability.

* int rb\_wait\_for\_single\_fd(int fd, int events, struct timeval \*timeout): Allows waiting on a single FD for one or multiple events with a specified timeout.
  
  `events` is a mask of any combination of the following values:
  
  * RB\_WAITFD\_IN - wait for readability of normal data
  * RB\_WAITFD\_OUT - wait for writability
  * RB\_WAITFD\_PRI - wait for readability of urgent data
  
  Use a NULL `timeout` to wait indefinitely.

#### I/O Multiplexing[](#io-multiplexing)

Ruby supports I/O multiplexing based on the select(2) system call. The Linux select\_tut(2) manpage <a href='http://man7.org/linux/man-pages/man2/select_tut.2.html' class='remote' target='_blank'>http://man7.org/linux/man-pages/man2/select_tut.2.html</a> provides a good overview on how to use select(2), and the Ruby API has analogous functions and data structures to the well-known select API. Understanding of select(2) is required to understand this section.

* typedef struct rb\_fdset\_t: The data structure which wraps the fd\_set bitmap used by select(2). This allows Ruby to use FD sets larger than that allowed by historic limitations on modern platforms.

* void rb\_fd\_init(rb\_fdset\_t *): Initializes the rb\_fdset\_t, it must be initialized before other rb\_fd\_* operations. Analogous to calling malloc(3) to allocate an fd\_set.

* void rb\_fd\_term(rb\_fdset\_t \*): Destroys the rb\_fdset\_t, releasing any memory and resources it used. It must be reinitialized using rb\_fd\_init before future use. Analogous to calling free(3) to release memory for an fd\_set.

* void rb\_fd\_zero(rb\_fdset\_t \*): Clears all FDs from the rb\_fdset\_t, analogous to FD\_ZERO(3).

* void rb\_fd\_set(int fd, rb\_fdset\_t \*): Adds a given FD in the rb\_fdset\_t, analogous to FD\_SET(3).

* void rb\_fd\_clr(int fd, rb\_fdset\_t \*): Removes a given FD from the rb\_fdset\_t, analogous to FD\_CLR(3).

* int rb\_fd\_isset(int fd, const rb\_fdset\_t \*): Returns true if a given FD is set in the rb\_fdset\_t, false if not. Analogous to FD\_ISSET(3).

* int rb\_thread\_fd\_select(int nfds, rb\_fdset\_t \*readfds, rb\_fdset\_t \*writefds, rb\_fdset\_t \*exceptfds, struct timeval \*timeout): Analogous to the select(2) system call, but allows other Ruby threads to be scheduled while waiting.
  
  When only waiting on a single FD, favor rb\_io\_wait\_readable, rb\_io\_wait\_writable, or rb\_wait\_for\_single\_fd functions since they can be optimized for specific platforms (currently, only Linux).

#### Initialize and Start the Interpreter[](#initialize-and-start-the-interpreter)

The embedding API functions are below (not needed for extension libraries):

* void ruby\_init(): Initializes the interpreter.

* void \*ruby\_options(int argc, char \*\*argv): Process command line arguments for the interpreter. And compiles the Ruby source to execute. It returns an opaque pointer to the compiled source or an internal special value.

* int ruby\_run\_node(void \*n): Runs the given compiled source and exits this process. It returns EXIT\_SUCCESS if successfully runs the source. Otherwise, it returns other value.

* void ruby\_script(char \*name): Specifies the name of the script ($0).

#### Hooks for the Interpreter Events[](#hooks-for-the-interpreter-events)

* void rb\_add\_event\_hook(rb\_event\_hook\_func\_t func, rb\_event\_flag\_t events, VALUE data): Adds a hook function for the specified interpreter events. events should be OR'ed value of:
  
  
  ```ruby
    RUBY_EVENT_LINE
    RUBY_EVENT_CLASS
    RUBY_EVENT_END
    RUBY_EVENT_CALL
    RUBY_EVENT_RETURN
    RUBY_EVENT_C_CALL
    RUBY_EVENT_C_RETURN
    RUBY_EVENT_RAISE
    RUBY_EVENT_ALL
  ```
  
  The definition of rb\_event\_hook\_func\_t is below:
  
  
  ```
    typedef void (*rb_event_hook_func_t)(rb_event_t event, VALUE data,
                                         VALUE self, ID id, VALUE klass)
  ```
  
  The third argument `data` to rb\_add\_event\_hook() is passed to the hook function as the second argument, which was the pointer to the current NODE in 1.8. See RB\_EVENT\_HOOKS\_HAVE\_CALLBACK\_DATA below.

* int rb\_remove\_event\_hook(rb\_event\_hook\_func\_t func): Removes the specified hook function.

#### Memory usage[](#memory-usage)

* void rb\_gc\_adjust\_memory\_usage(ssize\_t diff): Adjusts the amount of registered external memory. You can tell GC how much memory is used by an external library by this function. Calling this function with positive diff means the memory usage is increased; new memory block is allocated or a block is reallocated as larger size. Calling this function with negative diff means the memory usage is decreased; a memory block is freed or a block is reallocated as smaller size. This function may trigger the GC.

#### Macros for Compatibility[](#macros-for-compatibility)

Some macros to check API compatibilities are available by default.

* NORETURN\_STYLE\_NEW: Means that NORETURN macro is functional style instead of prefix.

* HAVE\_RB\_DEFINE\_ALLOC\_FUNC: Means that function rb\_define\_alloc\_func() is provided, that means the allocation framework is used. This is same as the result of have\_func("rb\_define\_alloc\_func", "ruby.h").

* HAVE\_RB\_REG\_NEW\_STR: Means that function rb\_reg\_new\_str() is provided, that creates Regexp object from String object. This is same as the result of have\_func("rb\_reg\_new\_str", "ruby.h").

* HAVE\_RB\_IO\_T: Means that type rb\_io\_t is provided.

* USE\_SYMBOL\_AS\_METHOD\_NAME: Means that Symbols will be returned as method names, e.g., `Module#methods`, #singleton\_methods and so on.

* HAVE\_RUBY\_\*\_H: Defined in ruby.h and means corresponding header is available. For instance, when HAVE\_RUBY\_ST\_H is defined you should use ruby/st.h not mere st.h.

* RB\_EVENT\_HOOKS\_HAVE\_CALLBACK\_DATA: Means that rb\_add\_event\_hook() takes the third argument `data`, to be passed to the given event hook function.

#### Defining backward compatible macros for keyword argument functions[](#defining-backward-compatible-macros-for-keyword-argument-functions)

Most ruby C extensions are designed to support multiple Ruby versions. In order to correctly support Ruby 2.7+ in regards to keyword argument separation, C extensions need to use `*_kw` functions. However, these functions do not exist in Ruby 2.6 and below, so in those cases macros should be defined to allow you to use the same code on multiple Ruby versions. Here are example macros you can use in extensions that support Ruby 2.6 (or below) when using the `*_kw` functions introduced in Ruby 2.7.


```
\#ifndef RB_PASS_KEYWORDS
/* Only define macros on Ruby <2.7 */
\#define rb_funcallv_kw(o, m, c, v, kw) rb_funcallv(o, m, c, v)
\#define rb_funcallv_public_kw(o, m, c, v, kw) rb_funcallv_public(o, m, c, v)
\#define rb_funcall_passing_block_kw(o, m, c, v, kw) rb_funcall_passing_block(o, m, c, v)
\#define rb_funcall_with_block_kw(o, m, c, v, b, kw) rb_funcall_with_block(o, m, c, v, b)
\#define rb_scan_args_kw(kw, c, v, s, ...) rb_scan_args(c, v, s, __VA_ARGS__)
\#define rb_call_super_kw(c, v, kw) rb_call_super(c, v)
\#define rb_yield_values_kw(c, v, kw) rb_yield_values2(c, v)
\#define rb_yield_splat_kw(a, kw) rb_yield_splat(a)
\#define rb_block_call_kw(o, m, c, v, f, p, kw) rb_block_call(o, m, c, v, f, p)
\#define rb_fiber_resume_kw(o, c, v, kw) rb_fiber_resume(o, c, v)
\#define rb_fiber_yield_kw(c, v, kw) rb_fiber_yield(c, v)
\#define rb_enumeratorize_with_size_kw(o, m, c, v, f, kw) rb_enumeratorize_with_size(o, m, c, v, f)
\#define SIZED_ENUMERATOR_KW(obj, argc, argv, size_fn, kw_splat) \
    rb_enumeratorize_with_size((obj), ID2SYM(rb_frame_this_func()), \
                               (argc), (argv), (size_fn))
\#define RETURN_SIZED_ENUMERATOR_KW(obj, argc, argv, size_fn, kw_splat) do { \
        if (!rb_block_given_p())                                            \
            return SIZED_ENUMERATOR(obj, argc, argv, size_fn);              \
    } while (0)
\#define RETURN_ENUMERATOR_KW(obj, argc, argv, kw_splat) RETURN_SIZED_ENUMERATOR(obj, argc, argv, 0)
\#define rb_check_funcall_kw(o, m, c, v, kw) rb_check_funcall(o, m, c, v)
\#define rb_obj_call_init_kw(o, c, v, kw) rb_obj_call_init(o, c, v)
\#define rb_class_new_instance_kw(c, v, k, kw) rb_class_new_instance(c, v, k)
\#define rb_proc_call_kw(p, a, kw) rb_proc_call(p, a)
\#define rb_proc_call_with_block_kw(p, c, v, b, kw) rb_proc_call_with_block(p, c, v, b)
\#define rb_method_call_kw(c, v, m, kw) rb_method_call(c, v, m)
\#define rb_method_call_with_block_kw(c, v, m, b, kw) rb_method_call_with_block(c, v, m, b)
\#endif
```

### Appendix C. Functions available for use in extconf.rb[](#appendix-c-functions-available-for-use-in-extconfrb)

See documentation for mkmf.

### Appendix D. Generational GC[](#appendix-d-generational-gc)

Ruby 2.1 introduced a generational garbage collector (called RGenGC). RGenGC (mostly) keeps compatibility.

Generally, the use of the technique called write barriers is required in extension libraries for generational GC (https://en.wikipedia.org/wiki/Garbage_collection_%28computer_science%29). RGenGC works fine without write barriers in extension libraries.

If your library adheres to the following tips, performance can be further improved. Especially, the "Don't touch pointers directly" section is important.

#### Incompatibility[](#incompatibility)

You can't write RBASIC(obj)->klass field directly because it is const value now.

Basically you should not write this field because MRI expects it to be an immutable field, but if you want to do it in your extension you can use the following functions:

* VALUE rb\_obj\_hide(VALUE obj): Clear RBasic::klass field. The object will be an internal object. ObjectSpace::each\_object can't find this object.

* VALUE rb\_obj\_reveal(VALUE obj, VALUE klass): Reset RBasic::klass to be klass. We expect the `klass` is hidden class by rb\_obj\_hide().

#### Write barriers[](#write-barriers)

RGenGC doesn't require write barriers to support generational GC. However, caring about write barrier can improve the performance of RGenGC. Please check the following tips.

##### Don't touch pointers directly[](#dont-touch-pointers-directly)

In MRI (include/ruby/ruby.h), some macros to acquire pointers to the internal data structures are supported such as RARRAY\_PTR(), RSTRUCT\_PTR() and so on.

DO NOT USE THESE MACROS and instead use the corresponding C-APIs such as rb\_ary\_aref(), rb\_ary\_store() and so on.

##### Consider whether to insert write barriers[](#consider-whether-to-insert-write-barriers)

You don't need to care about write barriers if you only use built-in types.

If you support T\_DATA objects, you may consider using write barriers.

Inserting write barriers into T\_DATA objects only works with the following type objects: (a) long-lived objects, (b) when a huge number of objects are generated and (c) container-type objects that have references to other objects. If your extension provides such a type of T\_DATA objects, consider inserting write barriers.

(a): short-lived objects don't become old generation objects. (b): only a few oldgen objects don't have performance impact. (c): only a few references don't have performance impact.

Inserting write barriers is a very difficult hack, it is easy to introduce critical bugs. And inserting write barriers has several areas of overhead. Basically we don't recommend you insert write barriers. Please carefully consider the risks.

##### Combine with built-in types[](#combine-with-built-in-types)

Please consider utilizing built-in types. Most built-in types support write barrier, so you can use them to avoid manually inserting write barriers.

For example, if your T\_DATA has references to other objects, then you can move these references to Array. A T\_DATA object only has a reference to an array object. Or you can also use a Struct object to gather a T\_DATA object (without any references) and an that Array contains references.

With use of such techniques, you don't need to insert write barriers anymore.

##### Insert write barriers[](#insert-write-barriers)

\[AGAIN\] Inserting write barriers is a very difficult hack, and it is easy to introduce critical bugs. And inserting write barriers has several areas of overhead. Basically we don't recommend you insert write barriers. Please carefully consider the risks.

Before inserting write barriers, you need to know about RGenGC algorithm (gc.c will help you). Macros and functions to insert write barriers are available in include/ruby/ruby.h. An example is available in iseq.c.

For a complete guide for RGenGC and write barriers, please refer to <a href='https://bugs.ruby-lang.org/projects/ruby-trunk/wiki/RGenGC' class='remote' target='_blank'>https://bugs.ruby-lang.org/projects/ruby-trunk/wiki/RGenGC</a>.

### Appendix E. RB\_GC\_GUARD to protect from premature GC[](#appendix-e-rbgcguard-to-protect-from-premature-gc)

C Ruby currently uses conservative garbage collection, thus VALUE variables must remain visible on the stack or registers to ensure any associated data remains usable. Optimizing C compilers are not designed with conservative garbage collection in mind, so they may optimize away the original VALUE even if the code depends on data associated with that VALUE.

The following example illustrates the use of RB\_GC\_GUARD to ensure the contents of sptr remain valid while the second invocation of rb\_str\_new\_cstr is running.


```
VALUE s, w;
const char *sptr;

s = rb_str_new_cstr("hello world!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!");
sptr = RSTRING_PTR(s);
w = rb_str_new_cstr(sptr + 6); /* Possible GC invocation */

RB_GC_GUARD(s); /* ensure s (and thus sptr) do not get GC-ed */
```

In the above example, RB\_GC\_GUARD must be placed *after* the last use of sptr. Placing RB\_GC\_GUARD before dereferencing sptr would be of no use. RB\_GC\_GUARD is only effective on the VALUE data type, not converted C data types.

RB\_GC\_GUARD would not be necessary at all in the above example if non-inlined function calls are made on the `s` VALUE after sptr is dereferenced. Thus, in the above example, calling any un-inlined function on `s` such as:


```ruby
rb_str_modify(s);
```

Will ensure `s` stays on the stack or register to prevent a GC invocation from prematurely freeing it.

Using the RB\_GC\_GUARD macro is preferable to using the "volatile" keyword in C. RB\_GC\_GUARD has the following advantages:

1.  the intent of the macro use is clear

2.  RB\_GC\_GUARD only affects its call site, "volatile" generates some extra code every time the variable is used, hurting optimization.

3.  "volatile" implementations may be buggy/inconsistent in some compilers and architectures. RB\_GC\_GUARD is customizable for broken systems/compilers without negatively affecting other systems.



### MakeMakefile[](#makemakefile)

mkmf.rb is used by Ruby C extensions to generate a Makefile which will correctly compile and link the C extension to Ruby and a third-party library.

<a href='https://ruby-doc.org/stdlib-2.7.0/libdoc/mkmf/rdoc/MakeMakefile.html' class='ruby-doc remote' target='_blank'>MakeMakefile Reference</a>

