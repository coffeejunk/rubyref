# WIN32OLE

`WIN32OLE` objects represent OLE Automation object in Ruby.

By using WIN32OLE, you can access OLE server like VBScript.

Here is sample script.

    require 'win32ole'

    excel = WIN32OLE.new('Excel.Application')
    excel.visible = true
    workbook = excel.Workbooks.Add();
    worksheet = workbook.Worksheets(1);
    worksheet.Range("A1:D1").value = ["North","South","East","West"];
    worksheet.Range("A2:B2").value = [5.2, 10];
    worksheet.Range("C2").value = 8;
    worksheet.Range("D2").value = 20;

    range = worksheet.Range("A1:D2");
    range.select
    chart = workbook.Charts.Add;

    workbook.saved = true;

    excel.ActiveWorkbook.Close(0);
    excel.Quit();

Unfortunately, Win32OLE doesn't support the argument passed by reference
directly. Instead, Win32OLE provides WIN32OLE::ARGV or WIN32OLE_VARIANT
object. If you want to get the result value of argument passed by reference,
you can use WIN32OLE::ARGV or WIN32OLE_VARIANT.

    oleobj.method(arg1, arg2, refargv3)
    puts WIN32OLE::ARGV[2]   # the value of refargv3 after called oleobj.method

or

    refargv3 = WIN32OLE_VARIANT.new(XXX,
                WIN32OLE::VARIANT::VT_BYREF|WIN32OLE::VARIANT::VT_XXX)
    oleobj.method(arg1, arg2, refargv3)
    p refargv3.value # the value of refargv3 after called oleobj.method.

[WIN32OLE Reference](https://ruby-doc.org/stdlib-2.7.0/libdoc/win32ole/rdoc/WIN32OLE.html)
