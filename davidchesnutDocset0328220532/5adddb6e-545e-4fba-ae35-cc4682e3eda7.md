
# How to: Retrieve a list of the hidden rows or columns in a spreadsheet document (Open XML SDK)
***Applies to:** Office 2013 | Open XML*

To use the sample code in this topic, you must install the [Open XML SDK
2.5](http://www.microsoft.com/en-us/download/details.aspx?id=30425). You
must explicitly reference the following assemblies in your project:

-   WindowsBase

-   DocumentFormat.OpenXml (installed by the Open XML SDK)

You must also use the following **using**
directives or **Imports** statements to compile
the code in this topic.

C\# 

    using System;
    using System.Collections.Generic;
    using System.IO;
    using System.Linq;
    using DocumentFormat.OpenXml.Packaging;
    using DocumentFormat.OpenXml.Spreadsheet;

Visual Basic 

    Imports System.IO
    Imports DocumentFormat.OpenXml.Packaging
    Imports DocumentFormat.OpenXml.Spreadsheet

## GetHiddenRowsOrCols Method

---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

You can use the **GetHiddenRowsOrCols** method
to retrieve a list of the hidden rows or columns in a worksheet. The
**GetHiddenRowsOrCols** method accepts three
parameters, indicating the following:

-   The name of the document to examine (string).

-   The name of the sheet to examine (string).

-   Whether to detect rows (true) or columns (false) (Boolean).

C\# 

    public static List<uint> GetHiddenRowsOrCols(
      string fileName, string sheetName, bool detectRows)

Visual Basic 

    Public Function GetHiddenRowsOrCols(
      ByVal fileName As String, ByVal sheetName As String,
      ByVal detectRows As Boolean) As List(Of UInteger)

## Calling the GetHiddenRowsOrCols Method

---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

The method returns a list of unsigned integers that contain each index
for the hidden rows or columns, if the specified worksheet contains any
hidden rows or columns (rows and columns are numbered starting at 1,
rather than 0.) To call the method, pass all the parameter values, as
shown in the following example code.

C\# 

    const string fileName = @"C:\users\public\documents\RetrieveHiddenRowsCols.xlsx";
    List<uint> items = GetHiddenRowsOrCols(fileName, "Sheet1", true);
    var sw = new StringWriter();
    foreach (var item in items)
        sw.WriteLine(item);
    Console.WriteLine(sw.ToString());

Visual Basic 

    Const fileName As String = "C:\Users\Public\Documents\RetrieveHiddenRowsCols.xlsx"
    Dim items As List(Of UInteger) =
        GetHiddenRowsOrCols(fileName, "Sheet1", True)
    Dim sw As New StringWriter
    For Each item In items
        sw.WriteLine(item)
    Next
    Console.WriteLine(sw.ToString())

## How the Code Works

-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

The code starts by creating a variable, <span
class="code">itemList</span>, that will contain the return value.

C\# 

    List<uint> itemList = new List<uint>();

Visual Basic 

    Dim itemList As New List(Of UInteger)

Next, the code opens the document, by using the <span sdata="cer"
target="M:DocumentFormat.OpenXml.Packaging.SpreadsheetDocument.Open(System.String,System.Boolean)"><span
class="nolink">SpreadsheetDocument.Open</span></span> method and
indicating that the document should be open for read-only access (the
final <span class="code">false</span> parameter value). Next the code
retrieves a reference to the workbook part, by using the <span
sdata="cer"
target="P:DocumentFormat.OpenXml.Packaging.SpreadsheetDocument.WorkbookPart"><span
class="nolink">WorkbookPart</span></span> property of the document.

C\# 

    using (SpreadsheetDocument document =
        SpreadsheetDocument.Open(fileName, false))
    {
        WorkbookPart wbPart = document.WorkbookPart;
        // Code removed here...
    }

Visual Basic 

    Using document As SpreadsheetDocument =
        SpreadsheetDocument.Open(fileName, False)

        Dim wbPart As WorkbookPart = document.WorkbookPart
        ' Code removed here...
    End Using

To find the hidden rows or columns, the code must first retrieve a
reference to the specified sheet, given its name. This is not as easy as
you might think. The code must look through all the sheet-type
descendants of the workbook part's <span sdata="cer"
target="P:DocumentFormat.OpenXml.Packaging.WorkbookPart.Workbook"><span
class="nolink">Workbook</span></span> property, examining the <span
sdata="cer"
target="P:DocumentFormat.OpenXml.Spreadsheet.Sheet.Name"><span
class="nolink">Name</span></span> property of each sheet that it finds.
Note that this search simply looks through the relations of the
workbook, and does not actually find a worksheet part. It simply finds a
reference to a <span sdata="cer"
target="T:DocumentFormat.OpenXml.Spreadsheet.Sheet"><span
class="nolink">Sheet</span></span> object, which contains information
such as the name and <span sdata="cer"
target="P:DocumentFormat.OpenXml.Spreadsheet.Sheet.Id"><span
class="nolink">Id</span></span> property of the sheet. The simplest way
to accomplish this is to use a LINQ query.

C\# 

    Sheet theSheet = wbPart.Workbook.Descendants<Sheet>().
        Where((s) => s.Name == sheetName).FirstOrDefault();
    if (theSheet == null)
    {
        throw new ArgumentException("sheetName");
    }

Visual Basic 

    Dim theSheet As Sheet = wbPart.Workbook.Descendants(Of Sheet)().
        Where(Function(s) s.Name = sheetName).FirstOrDefault()
    If theSheet Is Nothing Then
        Throw New ArgumentException("sheetName")

The <span sdata="cer"
target="M:System.Linq.Enumerable.FirstOrDefault``1(System.Collections.Generic.IEnumerable{``0},System.Func{``0,System.Boolean})">[FirstOrDefault](http://msdn2.microsoft.com/EN-US/library/bb358452)</span>
method returns either the first matching reference (a sheet, in this
case) or a null reference if no match was found. The code checks for the
null reference, and throws an exception if you passed in an invalid
sheet name. Now that you have information about the sheet, the code must
retrieve a reference to the corresponding worksheet part. The sheet
information you already retrieved provides an <span
class="keyword">Id</span> property, and given that <span
class="keyword">Id</span> property, the code can retrieve a reference to
the corresponding <span sdata="cer"
target="P:DocumentFormat.OpenXml.Spreadsheet.Worksheet.WorksheetPart"><span
class="nolink">WorksheetPart</span></span> property by calling the <span
sdata="cer"
target="M:DocumentFormat.OpenXml.Packaging.OpenXmlPartContainer.GetPartById(System.String)"><span
class="nolink">GetPartById</span></span> method of the <span sdata="cer"
target="T:DocumentFormat.OpenXml.Packaging.WorkbookPart"><span
class="nolink">WorkbookPart</span></span> object.

C\# 

    else
    {
        // The sheet does exist.
        WorksheetPart wsPart =
            (WorksheetPart)(wbPart.GetPartById(theSheet.Id));
        Worksheet ws = wsPart.Worksheet;
        // Code removed here...
    }

Visual Basic 

    Else
        ' The sheet does exist.
        Dim wsPart As WorksheetPart =
            CType(wbPart.GetPartById(theSheet.Id), WorksheetPart)
        Dim ws As Worksheet = wsPart.Worksheet
        ' Code removed here...
    End If

## Retrieving the List of Hidden Row or Column Index Values

---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

The code uses the <span class="code">detectRows</span> parameter that
you specified when you called the method to determine whether to
retrieve information about rows or columns.

C\# 

    if (detectRows)
    {
        // Retrieve hidden rows.
        // Code removed here...
    }
    else
    {
        // Retrieve hidden columns.
        // Code removed here...
    }

Visual Basic 

    If detectRows Then
        ' Retrieve hidden rows.
        ' Code removed here...
    Else
        ' Retrieve hidden columns.
        ' Code removed here...
    End If

The code that actually retrieves the list of hidden rows requires only a
single line of code.

C\# 

    itemList = ws.Descendants<Row>().
        Where((r) => r.Hidden != null && r.Hidden.Value).
        Select(r => r.RowIndex.Value).ToList<uint>();

Visual Basic 

    itemList = ws.Descendants(Of Row).
        Where(Function(r) r.Hidden IsNot Nothing AndAlso
              r.Hidden.Value).
        Select(Function(r) r.RowIndex.Value).ToList()

This single line accomplishes a lot, however. It starts by calling the
<span sdata="cer"
target="M:DocumentFormat.OpenXml.OpenXmlElement.Descendants"><span
class="nolink">Descendants</span></span> method of the worksheet,
retrieving a list of all the rows. The <span sdata="cer"
target="Overload:System.Linq.Enumerable.Where">[Where](http://msdn2.microsoft.com/EN-US/library/bb301979)</span>
method limits the results to only those rows where the <span sdata="cer"
target="P:DocumentFormat.OpenXml.Spreadsheet.Row.Hidden"><span
class="nolink">Hidden</span></span> property of the item is not null and
the value of the **Hidden** property is <span
class="keyword">True</span>. The <span sdata="cer"
target="Overload:System.Linq.Enumerable.Select">[Select](http://msdn2.microsoft.com/EN-US/library/bb357126)</span>
method projects the return value for each row, returning the value of
the <span sdata="cer"
target="P:DocumentFormat.OpenXml.Spreadsheet.Row.RowIndex"><span
class="nolink">RowIndex</span></span> property. Finally, the <span
sdata="cer"
target="M:System.Linq.Enumerable.ToList``1(System.Collections.Generic.IEnumerable{``0})">[ToList\<TSource\>](http://msdn2.microsoft.com/EN-US/library/bb342261)</span>
method converts the resulting <span sdata="cer"
target="T:System.Collections.Generic.IEnumerable`1">[IEnumerable\<T\>](http://msdn2.microsoft.com/EN-US/library/9eekhta0)</span>
interface into a <span sdata="cer"
target="T:System.Collections.Generic.List`1">[List\<T\>](http://msdn2.microsoft.com/EN-US/library/6sh2ey19)</span>
object of unsigned integers. If there are no hidden rows, the returned
list is empty.

Retrieving the list of hidden columns is a bit trickier, because Excel
collapses groups of hidden columns into a single element, and provides
<span sdata="cer"
target="P:DocumentFormat.OpenXml.Spreadsheet.Column.Min"><span
class="nolink">Min</span></span> and <span sdata="cer"
target="P:DocumentFormat.OpenXml.Spreadsheet.Column.Max"><span
class="nolink">Max</span></span> properties that describe the first and
last columns in the group. Therefore, the code that retrieves the list
of hidden columns starts the same as the code that retrieves hidden
rows. However, it must iterate through the index values (looping through
each item in the collection of hidden columns, adding each index from
the **Min** to the <span
class="keyword">Max</span> value, inclusively).

C\# 

    var cols = ws.Descendants<Column>().
      Where((c) => c.Hidden != null && c.Hidden.Value);
    foreach (Column item in cols)
    {
        for (uint i = item.Min.Value; i <= item.Max.Value; i++)
        {
            itemList.Add(i);
        }
    }

Visual Basic 

    Dim cols = ws.Descendants(Of Column).
      Where(Function(c) c.Hidden IsNot Nothing AndAlso
              c.Hidden.Value)
    For Each item As Column In cols
        For i As UInteger = item.Min.Value To item.Max.Value
            itemList.Add(i)
        Next
    Next

## Sample Code

------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

The following is the complete <span
class="keyword">GetHiddenRowsOrCols</span> code sample in C\# and Visual
Basic.

C\# 

    public static List<uint> GetHiddenRowsOrCols(
      string fileName, string sheetName, bool detectRows)
    {
        // Given a workbook and a worksheet name, return 
        // either a list of hidden row numbers, or a list 
        // of hidden column numbers. If detectRows is true, return
        // hidden rows. If detectRows is false, return hidden columns. 
        // Rows and columns are numbered starting with 1.

        List<uint> itemList = new List<uint>();

        using (SpreadsheetDocument document =
            SpreadsheetDocument.Open(fileName, false))
        {
            WorkbookPart wbPart = document.WorkbookPart;

            Sheet theSheet = wbPart.Workbook.Descendants<Sheet>().
                Where((s) => s.Name == sheetName).FirstOrDefault();
            if (theSheet == null)
            {
                throw new ArgumentException("sheetName");
            }
            else
            {
                // The sheet does exist.
                WorksheetPart wsPart =
                    (WorksheetPart)(wbPart.GetPartById(theSheet.Id));
                Worksheet ws = wsPart.Worksheet;

                if (detectRows)
                {
                    // Retrieve hidden rows.
                    itemList = ws.Descendants<Row>().
                        Where((r) => r.Hidden != null && r.Hidden.Value).
                        Select(r => r.RowIndex.Value).ToList<uint>();
                }
                else
                {
                    // Retrieve hidden columns.
                    var cols = ws.Descendants<Column>().
                        Where((c) => c.Hidden != null && c.Hidden.Value);
                    foreach (Column item in cols)
                    {
                        for (uint i = item.Min.Value; i <= item.Max.Value; i++)
                        {
                            itemList.Add(i);
                        }
                    }
                }
            }
        }
        return itemList;
    }

Visual Basic 

    Public Function GetHiddenRowsOrCols(
      ByVal fileName As String, ByVal sheetName As String,
      ByVal detectRows As Boolean) As List(Of UInteger)

        ' Given a workbook and a worksheet name, return either 
        ' a list of hidden row numbers, or a list of hidden 
        ' column numbers. If detectRows is True, return
        ' hidden rows. If detectRows is False, return hidden columns. 
        ' Rows and columns are numbered starting with 1.

        Dim itemList As New List(Of UInteger)

        Using document As SpreadsheetDocument =
            SpreadsheetDocument.Open(fileName, False)

            Dim wbPart As WorkbookPart = document.WorkbookPart

            Dim theSheet As Sheet = wbPart.Workbook.Descendants(Of Sheet)().
                Where(Function(s) s.Name = sheetName).FirstOrDefault()
            If theSheet Is Nothing Then
                Throw New ArgumentException("sheetName")
            Else
                ' The sheet does exist.
                Dim wsPart As WorksheetPart =
                    CType(wbPart.GetPartById(theSheet.Id), WorksheetPart)
                Dim ws As Worksheet = wsPart.Worksheet

                If detectRows Then
                    ' Retrieve hidden rows.
                    itemList = ws.Descendants(Of Row).
                        Where(Function(r) r.Hidden IsNot Nothing AndAlso
                              r.Hidden.Value).
                        Select(Function(r) r.RowIndex.Value).ToList()
                Else
                    ' Retrieve hidden columns.
                    Dim cols = ws.Descendants(Of Column).
                        Where(Function(c) c.Hidden IsNot Nothing AndAlso
                              c.Hidden.Value)
                    For Each item As Column In cols
                        For i As UInteger = item.Min.Value To item.Max.Value
                            itemList.Add(i)
                        Next
                    Next
                End If
            End If
        End Using
        Return itemList
    End Function

## See also

-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

#### Other resources

[Open XML SDK 2.5 class library
reference](http://msdn.microsoft.com/library/36c8a76e-ce1b-5959-7e85-5d77db7f46d6(Office.15).aspx)




