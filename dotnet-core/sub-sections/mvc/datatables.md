# Lets take a look at making your Paginated Tables in MVC Easier

## A Little Background

Pagination is an essential feature to any list of data. Databases can hold millions of records at any given time and as an Admin user you want the ability to easily filter, sort and paginate this data. 

As the developer however, this can prove challenging depending on the framework decided upon. For example Angular 9 provides this framework out of the box. And if you're about making things pretty, Angular Material has your back. However, from the MVC side, we are not so fortunate. Assuming you are writing your admin portal from scratch using Razor pages, adding tables can be an absolute nightmare.

This leads us to the problem.

## The Problem

Data can take any shape and can differ from table to table. One might one 3 or more tables on a single Razor page. This can easily lead to a misplaced `html` tag or worse... `CTRL + C, CTRL + V` for everything. The horror.

## The Solution

To make a methodology for successfully adding paginated tables to a Razor page with minimal code, that is completely customizable and simple to understand.

> **NOTE:** The solution below is not a creation of my own and is heavily adapted out of NopCommerce 4.20

## Technologies Used Above and Beyond DotNet Core MVC

- **<u>`Datatables.js`</u>** 
  - The leading open source software solution for paginated tables on the web. Find out more [here](https://datatables.net)
- **`jQuery.js`**, **`font-awesome`**, **`bootstrap-4`**, **`Foundation`**
  - Really not gonna explain these. (Google it)

---

## Step One - Defining The Overall Models

Comments will be in the code to explain individual fields

### `DataTablesModel.cs`

This is the base model that will hold all necessary information for constructing a `datatable` object

```c#
 public partial class DataTablesModel
 {
     protected const string DEFAULT_PAGING_TYPE = "simple_numbers";

     /// <summary>
     /// Ctor
     /// </summary>
     public DataTablesModel()
     {
         //set default values
         Info = true;
         RefreshButton = true;
         ServerSide = true;
         Processing = true;
         Paging = true;
         PagingType = DEFAULT_PAGING_TYPE;

         Filters = new List<FilterParameter>();
         ColumnCollection = new List<ColumnProperty>();
     }

     /// <summary>
     /// Gets or sets table name
     /// </summary>
     public string Name { get; set; }

     /// <summary>
     /// Gets or sets URL for data read (ajax)
     /// </summary>
     public DataUrl UrlRead { get; set; }

     /// <summary>
     /// Gets or sets URL for delete action (ajax)
     /// </summary>
     public DataUrl UrlDelete { get; set; }

     /// <summary>
     /// Gets or sets URL for update action (ajax)
     /// </summary>
     public DataUrl UrlUpdate { get; set; }

     /// <summary>
     /// Gets or sets search button Id
     /// </summary>
     public string SearchButtonId { get; set; }

     /// <summary>
     /// Gets or set filters controls
     /// </summary>
     public IList<FilterParameter> Filters { get; set; }

     /// <summary>
     /// Gets or sets data for table (ajax, json, array)
     /// </summary>
     public object Data { get; set; }

     /// <summary>
     /// Enable or disable the display of a 'processing' indicator when the table is being processed 
     /// </summary>
     public bool Processing { get; set; }

     /// <summary>
     /// Feature control DataTables' server-side processing mode
     /// </summary>
     public bool ServerSide { get; set; }

     /// <summary>
     /// Enable or disable table pagination.
     /// </summary>
     public bool Paging { get; set; }

     /// <summary>
     /// Enable or disable information ("1 to n of n entries")
     /// </summary>
     public bool Info { get; set; }

     /// <summary>
     /// Enable or disable refresh button
     /// </summary>
     public bool RefreshButton { get; set; }

     /// <summary>
     /// Pagination button display options.
     /// </summary>
     public string PagingType { get; set; }

     /// <summary>
     /// Number of rows to display on a single page when using pagination
     /// </summary>
     public int Length { get; set; }

     /// <summary>
     /// This parameter allows you to readily specify the entries in the length drop down select list that DataTables shows when pagination is enabled
     /// </summary>
     public string LengthMenu { get; set; }

     /// <summary>
     /// Indicates where particular features appears in the DOM
     /// </summary>
     public string Dom { get; set; }

     /// <summary>
     /// Feature control ordering (sorting) abilities in DataTables
     /// </summary>
     public bool Ordering { get; set; }

     /// <summary>
     /// Gets or sets custom render header function name(js)
     /// See also https://datatables.net/reference/option/headerCallback
     /// </summary>
     public string HeaderCallback { get; set; }

     /// <summary>
     /// Gets or sets a number of columns to generate in a footer. Set 0 to disable footer
     /// </summary>
     public int FooterColumns { get; set; }

     /// <summary>
     /// Gets or sets custom render footer function name(js)
     /// See also https://datatables.net/reference/option/footerCallback
     /// </summary>
     public string FooterCallback { get; set; }

     /// <summary>
     /// Gets or sets indicate of child table
     /// </summary>
     public bool IsChildTable { get; set; }

     /// <summary>
     /// Gets or sets child table
     /// </summary>
     public DataTablesModel ChildTable { get; set; }

     /// <summary>
     /// Gets or sets primary key column name for parent table
     /// </summary>
     public string PrimaryKeyColumn { get; set; }

     /// <summary>
     /// Gets or sets bind column name for delete action. If this field is not specified, the default will be the alias "id" for the delete action
     /// </summary>
     public string BindColumnNameActionDelete { get; set; }

     /// <summary>
     /// Gets or set column collection 
     /// </summary>
     public IList<ColumnProperty> ColumnCollection { get; set; }
 }
```



### `DataUrl.cs`

This class is used to define the controller and action name where it would fetch data from.

```c#
public partial class DataUrl
{
    #region Ctor

        /// <summary>
        /// Initializes a new instance of the DataUrl class 
        /// </summary>
        /// <param name="actionName">Action name</param>
        /// <param name="controllerName">Controller name</param>
        /// <param name="routeValues">Route values</param>
        public DataUrl(string actionName, string controllerName, RouteValueDictionary routeValues)
    {
        ActionName = actionName;
        ControllerName = controllerName;
        RouteValues = routeValues;
    }

    /// <summary>
    /// Initializes a new instance of the DataUrl class 
    /// </summary>
    /// <param name="url">URL</param>
    public DataUrl(string url)
    {
        Url = url;
    }

    /// <summary>
    /// Initializes a new instance of the DataUrl class 
    /// </summary>
    /// <param name="url">URL</param>
    /// <param name="dataId">Name of the column whose value is to be used as identifier in URL</param>
    public DataUrl(string url, string dataId)
    {
        Url = url;
        DataId = dataId;
    }

    /// <summary>
    /// Initializes a new instance of the DataUrl class 
    /// </summary>
    /// <param name="url">URL</param>
    /// <param name="trimEnd">Parameter indicating that you need to delete all occurrences of the character "/" at the end of the line</param>
    public DataUrl(string url, bool trimEnd)
    {
        Url = url;
        TrimEnd = trimEnd;
    }

    #endregion

    #region Properties

    /// <summary>
    /// Gets or sets the name of the action.
    /// </summary>
    public string ActionName { get; set; }

    /// <summary>
    /// Gets or sets the name of the controller.
    /// </summary>
    public string ControllerName { get; set; }

    /// <summary>
    /// Gets or sets the URL.
    /// </summary>
    public string Url { get; set; }

    /// <summary>
    /// Gets or sets the route values.
    /// </summary>
    public RouteValueDictionary RouteValues { get; set; }

    /// <summary>
    /// Gets or sets data Id
    /// </summary>
    public string DataId { get; set; }

    /// <summary>
    /// Gets or sets parameter indicating that you need to delete all occurrences of the character "/" at the end of the line
    /// </summary>
    public bool TrimEnd { get; set; }

    #endregion
}
```

### `EditType.cs`

This `enum` allows us to identify the column type. Eg Checkbox, String or Number

```c#
/// <summary>
/// Represents type editing of column
/// </summary>
public enum EditType
{
    Number = 1,
    Checkbox = 2,
    String = 3
}
```

### `FilterParameter.cs`

Allows us to specify a type of Filter for the data. For example: *Filter by Name*

```c#
/// <summary>
/// Represent DataTables filter parameter
/// </summary>
public partial class FilterParameter
{
    #region Ctor

        /// <summary>
        /// Initializes a new instance of the FilterParameter class by default as string type parameter
        /// </summary>
        /// <param name="name">Filter parameter name</param>
        public FilterParameter(string name)
    {
        Name = name;
        Type = typeof(string);
    }

    /// <summary>
    /// Initializes a new instance of the FilterParameter class
    /// </summary>
    /// <param name="name">Filter parameter name</param>
    /// <param name="modelName">Filter parameter model name</param>
    public FilterParameter(string name, string modelName)
    {
        Name = name;
        ModelName = modelName;
        Type = typeof(string);
    }

    /// <summary>
    /// Initializes a new instance of the FilterParameter class
    /// </summary>
    /// <param name="name">Filter parameter name</param>
    /// <param name="type">Filter parameter type</param>
    public FilterParameter(string name, Type type)
    {
        Name = name;
        Type = type;
    }

    /// <summary>
    /// Initializes a new instance of the FilterParameter class
    /// </summary>
    /// <param name="name">Filter parameter name</param>
    /// <param name="value">Filter parameter value</param>
    public FilterParameter(string name, object value)
    {
        Name = name;
        Type = value.GetType();
        Value = value;
    }

    /// <summary>
    /// Initializes a new instance of the FilterParameter class for linking "parent-child" tables
    /// </summary>
    /// <param name="name">Filter parameter name</param>
    /// <param name="parentName">Filter parameter parent name</param>
    /// <param name="isParentChildParameter">Parameter indicator for linking "parent-child" tables</param>
    public FilterParameter(string name, string parentName, bool isParentChildParameter = true)
    {
        Name = name;
        ParentName = parentName;
        Type = typeof(string);
    }

    #endregion

        #region Properties

        /// <summary>
        /// Filter field name
        /// </summary>
        public string Name { get; }

    /// <summary>
    /// Filter model name
    /// </summary>
    public string ModelName { get; }

    /// <summary>
    /// Filter field type
    /// </summary>
    public Type Type { get; }

    /// <summary>
    /// Filter field value
    /// </summary>
    public object Value { get; set; }

    /// <summary>
    /// Filter field parent name
    /// </summary>
    public string ParentName { get; set; }

    #endregion
}
```

### `ColumnProperty.cs`

This class allows us to define all the information we need to for a specific column in the table we want to create. 

```c#
public partial class ColumnProperty
{
    #region Ctor

    /// <summary>
    /// Initializes a new instance of the ColumnProperty class
    /// </summary>
    /// <param name="data">The data source for the column from the rows data object</param>
    public ColumnProperty(string data)
    {
        Data = data.ToCamelCase();
        //set default values
        Visible = true;
        Encode = true;
    }

    #endregion

    #region Properties

    /// <summary>
    /// Set the data source for the column from the rows data object / array.
    /// See also "https://datatables.net/reference/option/columns.data"
    /// </summary>
    public string Data { get; set; }

    /// <summary>
    /// Set the column title.
    /// </summary>
    public string Title { get; set; }

    /// <summary>
    /// Render (process) the data for use in the table. This property will modify the data that is used by DataTables for various operations.
    /// </summary>
    public IRender Render { get; set; }

    /// <summary>
    /// Column width assignment. This parameter can be used to define the width of a column, and may take any CSS value (3em, 20px etc).
    /// </summary>
    public string Width { get; set; }

    /// <summary>
    /// Column autowidth assignment. This can be disabled as an optimisation (it takes a finite amount of time to calculate the widths) if the tables widths are passed in using "width".
    /// </summary>
    public bool AutoWidth { get; set; }

    /// <summary>
    /// Indicate whether the column is master check box
    /// </summary>
    public bool IsMasterCheckBox { get; set; }

    /// <summary>
    /// Class to assign to each cell in the column.
    /// </summary>
    public string ClassName { get; set; }

    /// <summary>
    /// Enable or disable the display of this column.
    /// </summary>
    public bool Visible { get; set; }

    /// <summary>
    /// Enable or disable filtering on the data in this column.
    /// </summary>
    public bool Searchable { get; set; }

    /// <summary>
    /// Enable or disable editing on the data in this column.
    /// </summary>
    public bool Editable { get; set; }

    /// <summary>
    /// Data column type
    /// </summary>
    public EditType EditType { get; set; }

    /// <summary>
    /// Enable or disable encode on the data in this column.
    /// </summary>
    public bool Encode { get; set; }

    #endregion
}
```
We note in the constructor an extension method is used. `toCamelCase` This is to convert the name of the column in to a `javascript` friendly name. For more information on this type of extension refer to [My Personal Extension Methods - To Camel Case Extension](../extension-methods.md)

### `ButtonDefaults.cs`

This class, holding true to its name, is used to set some button style defaults. Yep.

```c#
/// <summary>
/// Button class name defaults
/// https://adminlte.io/themes/AdminLTE/pages/UI/buttons.html
/// </summary>
public static partial class ButtonDefaults
{
    /// <summary>
    /// Default class name
    /// </summary>
    public static string Default => "btn btn-default";

    /// <summary>
    /// Dark blue class name
    /// </summary>
    public static string Primary => "btn btn-primary";

    /// <summary>
    /// Green class name
    /// </summary>
    public static string Success => "btn btn-success";

    /// <summary>
    /// Blue class name
    /// </summary>
    public static string Info => "btn btn-info";

    /// <summary>
    /// Red class name
    /// </summary>
    public static string Danger => "btn btn-danger";

    /// <summary>
    /// Orange class name
    /// </summary>
    public static string Warning => "btn btn-warning";

    /// <summary>
    /// Olive class name
    /// </summary>
    public static string Olive => "btn bg-olive";
}
```

### `ColumnClassDefaults.cs`

Similar to the above, this class sets column class defaults

```c#
public static partial class ColumnClassDefaults
{
    /// <summary>
    /// Head and body content will be at center
    /// </summary>
    public static string CenterAll => "text-center";

    /// <summary>
    /// Parent-child control element
    /// </summary>
    public static string ChildControl => "child-control";

    /// <summary>
    /// Column contains button
    /// </summary>
    public static string Button => "button-column";
}
```

## Step Two - Defining Some Custom Renders

Renders are what sit in each cell within the table. Whether it be a date, button, Checkbox, You Name it. Due to the repetitive nature of this. They will not all be defined but the concept should be clear.

The following is the interface used as a definition.

```c#
/// <summary>
/// Represents render (process) the data for use in the DataTables.
/// </summary>
public partial interface IRender
{
}
```
The rest is some definitions:
```c#
/// <summary>
/// Represents boolean render for DataTables column
/// </summary>
public partial class RenderBoolean : IRender
{
}
/// <summary>
/// Represents button custom render for DataTables column
/// </summary>
public partial class RenderButtonCustom : IRender
{
    #region Ctor

        /// <summary>
        /// Initializes a new instance of the RenderButtonEdit class 
        /// </summary>
        /// <param name="className">Class name of button</param>
        /// <param name="title">Title button</param>
        public RenderButtonCustom(string className, string title)
    {
        ClassName = className;
        Title = title;
    }

    #endregion

        #region Properties

        /// <summary>
        /// Gets or sets Url to action
        /// </summary>
        public string Url { get; set; }

    /// <summary>
    /// Gets or sets button class name
    /// </summary>
    public string ClassName { get; set; }

    /// <summary>
    /// Gets or sets button title
    /// </summary>
    public string Title { get; set; }

    /// <summary>
    /// Gets or sets function name on click button
    /// </summary>
    public string OnClickFunctionName { get; set; }

    #endregion
}
/// <summary>
/// Represents button edit render for DataTables column
/// </summary>
public partial class RenderButtonEdit : IRender
{
    #region Ctor

        /// <summary>
        /// Initializes a new instance of the RenderButtonEdit class 
        /// </summary>
        /// <param name="url">URL to the edit action</param>
        public RenderButtonEdit(DataUrl url)
    {
        Url = url;
        ClassName = ButtonDefaults.Default;
    }

    #endregion

        #region Properties

        /// <summary>
        /// Gets or sets Url to action edit
        /// </summary>
        public DataUrl Url { get; set; }

    /// <summary>
    /// Gets or sets button class name
    /// </summary>
    public string ClassName { get; set; }

    #endregion
}
/// <summary>
/// Represents date render for DataTables column
/// </summary>
public partial class RenderDate : IRender
{
    #region Constants

        /// <summary>
        /// Default date format
        /// </summary>
        private string DEFAULT_DATE_FORMAT = "MM/DD/YYYY HH:mm:ss";

    #endregion

        #region Ctor

        public RenderDate()
    {
        //set default values
        Format = DEFAULT_DATE_FORMAT;
    }

    #endregion

        #region Properties

        /// <summary>
        /// Gets or sets format date (moment.js)
        /// See also "http://momentjs.com/"
        /// </summary>
        public string Format { get; set; }

    #endregion
}
/// <summary>
/// Represents picture render for DataTables column
/// </summary>
public partial class RenderPicture : IRender
{
    #region Ctor

        public RenderPicture(string srcPrefix = "")
    {
        SrcPrefix = srcPrefix;
    }

    #endregion

        #region Properties

        /// <summary>
        /// Gets or sets picture URL prefix
        /// </summary>
        public string SrcPrefix { get; set; }

    /// <summary>
    /// Gets or sets picture source
    /// </summary>
    public string Src { get; set; }

    #endregion
}
/// <summary>
/// Represents custom render for DataTables column
/// </summary>
public partial class RenderCustom : IRender
{
    #region Ctor

        /// <summary>
        /// Initializes a new instance of the RenderCustom class 
        /// </summary>
        /// <param name="functionName">Custom render function name that is used in js</param>
        public RenderCustom(string functionName)
    {
        FunctionName = functionName;
    }

    #endregion

        #region Properties

        /// <summary>
        /// Gets or sets custom render function name(js)
        /// See also https://datatables.net/reference/option/columns.render
        /// </summary>
        public string FunctionName { get; set; }

    #endregion
}
```

## Step Three - Lets Build The Partial Views Used to Generate This

There are three partial views that will be used to create this. 

1. `_Table.cshtml` - This is used to instantiate the table as well as build the structure around the table. For example
   1. The Page List Below the Table
   2. The Drop Down to choose how many rows to display
   3. The viewing item x of x 
2. `_Table.Definition.cshtml` - This is where the actual creation of the table occurs.
3. `_GridLocalization.cshtml` - Additional settings for `datatables.js`

These files will not be explained but just shown. To for detailed documentation on them [look here](https://dzone.com/articles/datatables-template-that-simplifies-the-work)

### `_Table.cshtml`

```html
@model DataTablesModel
@using System.Net
@functions
{ string GetUrl(DataUrl dataUrl)
    {
        return !string.IsNullOrEmpty(dataUrl?.ActionName) && !string.IsNullOrEmpty(dataUrl.ControllerName)
            ? Url.Action(dataUrl.ActionName, dataUrl.ControllerName, dataUrl.RouteValues)
            : !string.IsNullOrEmpty(dataUrl.Url)
            ? $"{(dataUrl.Url.StartsWith("~/", StringComparison.Ordinal) ? Url.Content(dataUrl.Url) : dataUrl.Url).TrimEnd('/')}" + (!dataUrl.TrimEnd ? "/" : "")
            : string.Empty;
    } }
<table class="table table-bordered table-hover table-striped dataTable" width="100%" id="@Model.Name">
    @if (Model.FooterColumns > 0)
    {
        //You need to add the footer before you create the table
        //as DataTables doesn't provide a method for creating a footer at the moment
<tfoot>
    <tr>
        @for (int i = 0; i < Model.FooterColumns; i++)
        {
<td></td>
       }
    </tr>
</tfoot>
}
</table>

@{ //check using MasterCheckBox
    var isMasterCheckBoxUsed = Model.ColumnCollection.Any(x => x.IsMasterCheckBox);
    //Model name for js function names
    var model_name = Model.Name.Replace("-", "_"); }

<script>
    @if (isMasterCheckBoxUsed)
    {
        //selectedIds - This variable will be used on views. It can not be renamed
        <text>
        var selectedIds = [];

        function updateMasterCheckbox() {
            var numChkBoxes = $('#@Model.Name input[type=checkbox][id!=mastercheckbox][class=checkboxGroups]').length;
            var numChkBoxesChecked = $('#@Model.Name input[type=checkbox][id!=mastercheckbox][class=checkboxGroups]:checked').length;
            $('#mastercheckbox').attr('checked', numChkBoxes == numChkBoxesChecked && numChkBoxes > 0);
        }
        </text>
    }

    function updateTable(tableSelector) {
        $(tableSelector).DataTable().ajax.reload();
        if (@isMasterCheckBoxUsed.ToString().ToLower()) {
            $('#mastercheckbox').attr('checked', false).change();
            selectedIds = [];
        }
    }

    $(document).ready(function () {
        $('#@Model.Name').DataTable({
             @await Html.PartialAsync("_Table.Definition", Model)
        });

        @if (!string.IsNullOrEmpty(Model.SearchButtonId))
        {
            <text>
            $('#@Model.SearchButtonId').click(function() {
                $('#@Model.Name').DataTable().ajax.reload();
                $('.checkboxGroups').attr('checked', false).change();
                selectedIds = [];
                return false;
            });
            </text>
        }
        @if (isMasterCheckBoxUsed)
        {
            <text>
            $('#mastercheckbox').click(function () {
                $('.checkboxGroups').attr('checked', $(this).is(':checked')).change();
            });

            $('#@Model.Name').on('change', 'input[type=checkbox][id!=mastercheckbox][class=checkboxGroups]', function (e) {
                var $check = $(this);
                var checked = jQuery.inArray($check.val(), selectedIds);
                if ($check.is(':checked') == true) {
                    if (checked == -1) {
                        selectedIds.push($check.val());
                    }
                } else if (checked > -1) {
                    selectedIds = $.grep(selectedIds, function (item, index) {
                        return item != $check.val();
                    });
                }
                updateMasterCheckbox();
            });
            </text>
        }
    });
</script>
@if ((Model.UrlDelete != null) || (Model.ChildTable?.UrlDelete != null))
{
<text>
    <script>
        function table_deletedata_@(model_name)(dataId) {
           if (confirm('Are you sure you would like to delete this')) {
                var postData = {
                @if (!string.IsNullOrEmpty(Model.BindColumnNameActionDelete))
                {
                    <text>
                    @Model.BindColumnNameActionDelete: dataId
                    </text>
                }
                else
                {
                    <text>
                    id: dataId
                    </text>
                }
                };
                addAntiForgeryToken(postData);

                $.ajax({
                    url: "@Html.Raw(GetUrl((Model.ChildTable?.UrlDelete != null) ? Model.ChildTable?.UrlDelete : Model.UrlDelete))",
                    type: "@WebRequestMethods.Http.Post",
                    dataType: "json",
                    data: postData,
                    success: function (data, textStatus, jqXHR) {
                        //display error if returned
                        if (data) {
                            display_nop_error(data);
                        }
                        //refresh grid
                        $('#@Model.Name').DataTable().draw(false);
                    },
                    error: function (jqXHR, textStatus, errorThrown) {
                        alert(errorThrown);
                    }
                });
            }
            else {
                return false;
            }
        }
    </script>
    </text>}

@if (Model.UrlUpdate != null || Model.ChildTable?.UrlUpdate != null)
{
<text>
    <script>
        var editIndexTable_@(model_name) = -1;
        var editRowData_@(model_name) = [];
        var columnData_@(model_name) = [];
        @foreach(var column in Model.ColumnCollection.Where(x => x.Editable))
        {
            <text>
                var obj = { 'Data': '@column.Data', 'Editable': @column.Editable.ToString().ToLower(), 'Type': '@column.EditType.ToString().ToLower()' }
                columnData_@(model_name).push(obj);
            </text>
        }

        function editData_@(model_name)(dataId, data) {
            var modData = data;
            if (typeof data == 'string') {
                modData = data.replace(/[.*+?^${}()|[\]\\]/g, "_");
            }
            $('#buttonEdit_@(model_name)' + modData).hide();
            $('#buttonConfirm_@(model_name)' + modData).show();
            $('#buttonCancel_@(model_name)' + modData).show();
            rowEditMode_@(model_name)(dataId);
        }

        function rowEditMode_@(model_name)(rowid) {
            var prevRow;
            var rowIndexVlaue = parseInt(rowid[0].rowIndex);
            if (editIndexTable_@(model_name) == -1) {
                saveRowIntoArray_@(model_name)(rowid);
                rowid.attr("editState", "editState");
                editIndexTable_@(model_name) = rowid.rowIndex;
                setEditStateValue_@(model_name)(rowid);
            }
            else {
                prevRow = $("[editState=editState]");
                prevRow.attr("editState", "");
                rowid.attr("editState", "editState");
                editIndexTable_@(model_name) = rowIndexVlaue;
                saveArrayIntoRow_@(model_name)(prevRow);
                saveRowIntoArray_@(model_name)(rowid);
                setEditStateValue_@(model_name)(rowid);
            }
        }

        function escapeQuotHtml (value) {
            return String(value).replace(/["]/g, function (s) {
                return '&quot;';
            });
        }

        function setEditStateValue_@(model_name)(curRow) {
            for (var cellName in editRowData_@(model_name)) {
                var columnType = 'string';

                $.each(columnData_@(model_name), function (index, element) {
                    if (element.Data == cellName) {
                        columnType = element.Type;
                    }
                });

                if (columnType == 'number') {
                    $($(curRow).children("[data-columnname='" + cellName + "']")[0]).html('<input value="' + editRowData_@(model_name)[cellName] + '" class="userinput" type="number" min="@int.MinValue" max="@int.MaxValue"/>');
                }
                if (columnType == 'checkbox') {
                    var cellValue = editRowData_@(model_name)[cellName];
                    if ($(cellValue).attr('nop-value') === 'true') {
                        $($(curRow).children("[data-columnname='" + cellName + "']")[0]).html('<input value="true" checked class="userinput" type="checkbox" onclick="checkBoxClick_@(model_name)(this)"/>');
                    }
                    else {
                        $($(curRow).children("[data-columnname='" + cellName + "']")[0]).html('<input value="false" class="userinput" type="checkbox" onclick="checkBoxClick_@(model_name)(this)"/>');
                    }
                }
                if (columnType == 'string') {
                    var strValue = editRowData_@(model_name)[cellName];
                    $($(curRow).children("[data-columnname='" + cellName + "']")[0]).html('<input value="' + escapeQuotHtml(strValue) + '" class="userinput"  style="width: 99% " />');
                }
            }
        }

        function checkBoxClick_@(model_name)(checkBox) {
            var input = $(checkBox);
            if ($(input).val() === 'true') {
                $(input).val('false');
                $(input).removeAttr('checked');
            } else {
                $(input).val('true');
                $(input).attr('checked', 'checked');
            }
        }

        function confirmEditData_@(model_name)(dataId, data, nameData) {
            var origData = data;
            var modData = data;
            if (typeof data == 'string') {
                modData = data.replace(/[.*+?^${}()|[\]\\]/g, "_");
            }
            $('#buttonEdit_@(model_name)' + modData).show();
            $('#buttonConfirm_@(model_name)' + modData).hide();
            $('#buttonCancel_@(model_name)' + modData).hide();

            updateRowData_@(model_name)(dataId, origData, nameData);
        }

        function updateRowData_@(model_name)(currentCells, data, nameData) {
            var updateRowData = [];
            updateRowData.push({ 'pname': nameData, 'pvalue': data });
            $.each(columnData_@(model_name), function (index, element) {
                if (element.Editable == true) {
                    updateRowData.push({
                        'pname': element.Data, 'pvalue': $($($(currentCells).children("[data-columnname='" + element.Data + "']")).children('input')[0]).val()
                    });
                }
            });
            var postData = {};
            for (i = 0; i < updateRowData.length; i++) {
                postData[updateRowData[i].pname] = updateRowData[i].pvalue;
            }
            var tokenInput = $('input[name=__RequestVerificationToken]').val();
            postData['__RequestVerificationToken'] = tokenInput;
            addAntiForgeryToken(postData);

            $.ajax({
                url: "@Html.Raw(GetUrl((Model.ChildTable?.UrlUpdate != null) ? Model.ChildTable?.UrlUpdate : Model.UrlUpdate))",
                type: "POST",
                dataType: "json",
                data: postData,
                success: function (data, textStatus, jqXHR) {
                    //display error if returned
                    if (data) {
                        display_nop_error(data);
                    }
                    //refresh grid
                     $('#@Model.Name').DataTable().draw(false);
                },
                error: function (jqXHR, textStatus, errorThrown) {
                    alert(errorThrown);
                }
            });
        }

        function cancelEditData_@(model_name)(dataId, data) {
            var modData = data;
            if (typeof data == 'string') {
                modData = data.replace(/[.*+?^${}()|[\]\\]/g, "_");
            }
            $('#buttonEdit_@(model_name)' + modData).show();
            $('#buttonConfirm_@(model_name)' + modData).hide();
            $('#buttonCancel_@(model_name)' + modData).hide();

            var prevRow = $("[editState=editState]");
            prevRow.attr("editState", "");
            if (prevRow.length > 0) {
                saveArrayIntoRow_@(model_name)($(prevRow));
            }
            editIndexTable_@(model_name) = -1;
        }

        function saveArrayIntoRow_@(model_name)(cureentCells) {
            for (var cellName in editRowData_@(model_name)) {
                $($(cureentCells).children("[data-columnname='" + cellName + "']")[0]).html(editRowData_@(model_name)[cellName]);
            }
        }

        function saveRowIntoArray_@(model_name)(cureentCells) {
            $.each(columnData_@(model_name), function (index, element) {
                if (element.Editable == true) {
                    var htmlVal = $($(cureentCells).children("[data-columnname='" + element.Data + "']")[0]).html();
                    editRowData_@(model_name)[element.Data] = htmlVal;
                }
            });
        }
    </script>
    </text>}

@if (Model.ChildTable != null)
{
<text>
    <script>
        function getchild_@(model_name)(d) {
            return '<table id="child' + d.@(Model.PrimaryKeyColumn) + '" class="table table-bordered table-hover dataTable" width="100%" style="padding-left:2%;"></table>';
        }
        $(document).ready(function () {
            // Add event listener for opening and closing childs
            $('#@Model.Name tbody').on('click', 'td.child-control', function () {
                var tr = $(this).closest('tr');
                var tdi = tr.find('i.fa');
                var row = $('#@Model.Name').DataTable().row(tr);

                if (row.child.isShown()) {
                    // This row is already open - close it
                    row.child.hide();
                    tr.removeClass('shown');
                    tdi.first().removeClass('fa-caret-down');
                    tdi.first().addClass('fa-caret-right');
                }
                else {
                    // Open this row
                    row.child(getchild_@(model_name)(row.data())).show();
                    var classid = '#child' + row.data().@(Model.PrimaryKeyColumn);
                    $(classid).DataTable({
                        @await Html.PartialAsync("_Table.Definition", Model.ChildTable)
                    }).draw;
                    tr.addClass('shown');
                    tdi.first().removeClass('fa-caret-right');
                    tdi.first().addClass(' fa-caret-down');
                }
            });
        });
    </script>
</text>
}
```

### `_Table.Definition.cshtml`

```html
@model DataTablesModel
@using System.Net;

@{
    //the locale which MomentJS should use - the default is en (English).
    var locale = "en";

    //Model name for js function names
    var model_name = Model.Name.Replace("-", "_");

    //dom
    var buttonsPanel = "";
    var infoPanel = "<'col-lg-4 col-xs-12'<'float-lg-right text-center'i>>";

    if (Model.RefreshButton && !Model.IsChildTable)
    {
        buttonsPanel = "<'col-lg-1 col-xs-12'<'float-lg-right text-center data-tables-refresh'B>>";
        infoPanel = "<'col-lg-3 col-xs-12'<'float-lg-right text-center'i>>";
    }

    var dom = "<'row'<'col-md-12't>>" +
              "<'row margin-t-5'" +
                "<'col-lg-5 col-xs-12'<'float-lg-left'p>>" +
                "<'col-lg-3 col-xs-12'<'text-center'l>>" +
                infoPanel +
                buttonsPanel +
              ">";

    if (!string.IsNullOrEmpty(Model.Dom))
    {
        dom = Model.Dom;
    }
}

@functions
{
    string GetUrl(DataUrl dataUrl)
    {
        return !string.IsNullOrEmpty(dataUrl?.ActionName) && !string.IsNullOrEmpty(dataUrl.ControllerName)
            ? Url.Action(dataUrl.ActionName, dataUrl.ControllerName, dataUrl.RouteValues)
            : !string.IsNullOrEmpty(dataUrl.Url)
            ? $"{(dataUrl.Url.StartsWith("~/", StringComparison.Ordinal) ? Url.Content(dataUrl.Url) : dataUrl.Url).TrimEnd('/')}" + (!dataUrl.TrimEnd ? "/" : "")
            : string.Empty;
    }
}

@if (!string.IsNullOrEmpty(Model.HeaderCallback))
{
    <text>
    headerCallback: function (tfoot, data, start, end, display) {
        return @(Model.HeaderCallback)(tfoot, data, start, end, display);
    },
    </text>
}
@if (!string.IsNullOrEmpty(Model.FooterCallback))
{
    <text>
    footerCallback: function (tfoot, data, start, end, display) {
        return @(Model.FooterCallback)(tfoot, data, start, end, display);
    },
    </text>
}
@if (Model.Processing)
{
    <text>
    processing: @Model.Processing.ToString().ToLower(),
    </text>
}
@if (Model.ServerSide)
{
    <text>
    serverSide: @Model.ServerSide.ToString().ToLower(),
    </text>
}
@if (Model.Data != null)
{
    <text>
    data: @Html.Raw(Model.Data.ToString()),
    </text>
}
else
{
    <text>
    ajax:
    {
        url: "@Html.Raw(GetUrl(Model.UrlRead))",
        type: "@WebRequestMethods.Http.Post",
        dataType: "json",
        dataSrc: "data",
        data: function(data) {
            @if (Model.Filters != null)
            {
                foreach (var filter in Model.Filters)
                {
                    if (filter.Type == typeof(string))
                    {
                        if (Model.IsChildTable && !string.IsNullOrEmpty(filter.ParentName))
                        {
                            <text>
                            data.@filter.Name = row.data().@filter.ParentName;
                            </text>
                            continue;
                        }

                        if (!string.IsNullOrEmpty(filter.ModelName))
                        {
                            <text>
                            data.@filter.Name = $('#@(filter.ModelName)_@filter.Name').val();
                            </text>
                        }
                        else
                        {
                            <text>
                            data.@filter.Name = $('#@filter.Name').val();
                            </text>
                        }
                        continue;
                    }
                    if (filter.Type == typeof(bool))
                    {
                        <text>
                        data.@filter.Name = $('#@filter.Name').is(':checked');
                        </text>
                        continue;
                    }
                    if (filter.Type == typeof(int))
                    {
                        if (int.TryParse(@filter.Value.ToString(), out int val))
                        {
                            <text>
                            data.@filter.Name = @val;
                            </text>
                        }
                        continue;
                    }
                }
            }
            addAntiForgeryToken(data);
            return data;
        }
    },
    </text>
}
scrollX: true,
info: @Model.Info.ToString().ToLower(),
paging: @Model.Paging.ToString().ToLower(),
pagingType: '@Model.PagingType',
language: @await Html.PartialAsync("_GridLocalization"),
pageLength: @Model.Length,
@if (!string.IsNullOrEmpty(Model.LengthMenu))
{
    <text>
        lengthMenu: [@Model.LengthMenu],
    </text>
}
else
{
    <text>
        lengthChange: false,
    </text>
}
ordering: @Model.Ordering.ToString().ToLower(),
@if (Model.RefreshButton)
{
    <text>
        buttons: [{
            name: 'refresh',
            text: '<i class="fa fa-refresh" style="padding-left: 5px"></i>',
            action: function () {
                updateTable('#@Model.Name');
            }
        }],
    </text>
}
dom: '@JavaScriptEncoder.Default.Encode(dom)',
columns: [    
    @for (int i = 0; i < Model.ColumnCollection.Count; i++)
    {
        var column = Model.ColumnCollection[i];
        <text>
        {            
            @if (!string.IsNullOrEmpty(column.Title) && !column.IsMasterCheckBox)
            {
                <text>
                title: '@JavaScriptEncoder.Default.Encode(column.Title)',
                </text>
            }
            else
            {
                if (!string.IsNullOrEmpty(column.Title) && column.IsMasterCheckBox)
                {
                    <text>
                    title: '<div class="checkbox"><label><input id="mastercheckbox" type="checkbox" />@JavaScriptEncoder.Default.Encode(column.Title)</label></div>',
                    </text>
                }
                else
                {
                    if (string.IsNullOrEmpty(column.Title) && column.IsMasterCheckBox)
                    {
                        <text>
                        title: '<input id="mastercheckbox" type="checkbox"/>',
                        </text>
                    }
                }
            }
            width: '@column.Width',
            visible: @column.Visible.ToString().ToLower(),
            searchable: @column.Searchable.ToString().ToLower(),
            @if (column.AutoWidth)
            {
                <text>
                autoWidth: @column.AutoWidth.ToString().ToLower(),
                </text>
            }
            @if (!string.IsNullOrEmpty(column.ClassName))
            {
                <text>
                className: '@column.ClassName',
                </text>
            }
            @if ((Model.UrlUpdate != null) || (Model.ChildTable?.UrlUpdate != null))
            {
                <text>
                createdCell: function (td, cellData, rowData, row, col) {
                   $(td).attr('data-columnname', '@column.Data');
                },
                </text>
            }
            @if (column.Encode && column.Render == null)
            {
                <text>
                render: $.fn.dataTable.render.text(),
                </text>
            }
            @switch (column.Render)
            {
                case RenderLink link:
                    <text>
                    render: function (data, type, row, meta) {
                        var textRenderer = $.fn.dataTable.render.text().display;
                        @if (!string.IsNullOrEmpty(link.Title))
                        {
                            <text>
                            return '<a href="@GetUrl(link.Url)' + textRenderer(row.@link.Url.DataId) + '">@JavaScriptEncoder.Default.Encode(link.Title)</a>';
                            </text>
                        }
                        else
                        {
                            <text>
                            return '<a href="@GetUrl(link.Url)' + textRenderer(row.@link.Url.DataId) + '">' + textRenderer(data) + '</a>';
                            </text>
                        }
                    },
                    </text>
                    break;
                case RenderDate date:
                    <text>
                    render: function (data, type, row, meta) {
                        return (data) ? moment(data).locale('@locale').format('@date.Format') : null;
                    },
                    </text>
                    break;
                case RenderButtonRemove button:
                    <text>
                    render: function (data, type, row, meta) {
                        return '<a href="#" class="@button.ClassName" onclick="table_deletedata_@(model_name)(\'' + data + '\');return false;"><i class="fa fa-remove"></i>@button.Title</a>';
                    },
                    </text>
                    break;
                    case RenderButtonsInlineEdit button:
                    <text>
                        render: function (data, type, row, meta) {
                            var origData = data;
                            var modData = data;
                            if (typeof data == 'string'){
                                modData = data.replace(/[.*+?^${}()|[\]\\]/g, "_");
                            }
                            return '<a href="#" class="@button.ClassName" id="buttonEdit_@(model_name)'+ modData + '" onclick="editData_@(model_name)($(this).parent().parent(), \'' + origData + '\');return false;"><i class="fa fa-pencil"></i>Edit</a>' +
                            '<a href="#" class="@button.ClassName" id="buttonConfirm_@(model_name)'+ modData + '" style="display:none" onclick="confirmEditData_@(model_name)($(this).parent().parent(), \'' + origData + '\', \'@column.Data\');return false;"><i class="fa fa-check"></i>Update</a>' +
                            '<a href="#" class="@button.ClassName" id="buttonCancel_@(model_name)'+ modData + '" style="display:none" onclick="cancelEditData_@(model_name)(\'' + row + '\', \'' + origData + '\');return false;"><i class="fa fa-ban"></i>Cancel</a>';
                        },                         
                    </text>
                    break;
                case RenderButtonEdit buttonEdit:
                    <text>
                    render: function (data, type, row, meta) {
                        return '<a class="@buttonEdit.ClassName" href="@GetUrl(buttonEdit.Url)' + data + '"><i class="fa fa-pencil"></i>Edit</a>';
                    },
                    </text>
                    break;
                case RenderButtonView buttonView:
                    <text>
                    render: function (data, type, row, meta) {
                        return '<a class="@buttonView.ClassName" href="@GetUrl(buttonView.Url)' + data + '"><i class="fa fa-eye"></i>View</a>';
                    },
                    </text>
                    break;
                case RenderButtonCustom buttonCustom:
                    if (!string.IsNullOrEmpty(buttonCustom.Url))
                    {
                        <text>
                        render: function (data, type, row, meta) {
                            return '<a class="@buttonCustom.ClassName" href="@buttonCustom.Url' + data + '">@JavaScriptEncoder.Default.Encode(buttonCustom.Title)</a>';
                        },
                        </text>
                    }
                    if (!string.IsNullOrEmpty(buttonCustom.OnClickFunctionName))
                    {
                        <text>
                        render: function (data, type, row, meta) {
                            return '<a class="@buttonCustom.ClassName" onclick="@buttonCustom.OnClickFunctionName' + '(' + data + ');">@JavaScriptEncoder.Default.Encode(buttonCustom.Title)</a>';
                        },
                        </text>
                    }
                    break;
                case RenderPicture picture:
                    <text>
                    render: function (data, type, row, meta) {
                        @if (!string.IsNullOrEmpty(picture.Src))
                        {
                            <text>
                            return '<img src="@(picture.SrcPrefix)@(picture.Src)"/>';
                            </text>
                        }
                        else
                        {
                            <text>
                            return '<img src="@(picture.SrcPrefix)' + data + '"/>';
                            </text>
                        }
                    },
                    </text>
                    break;
                case RenderCheckBox checkBox:
                    <text>
                    render: function (data, type, row, meta) {
                        return (data === 'true')
                            ? '<input name="@checkBox.Name" value="' + row.Id + '" type="checkbox" class="checkboxGroups" checked="checked" />'
                            : '<input name="@checkBox.Name" value="' + row.Id + '" type="checkbox" class="checkboxGroups" />';
                    },
                    </text>
                    break;
                case RenderBoolean renderBoolean:
                    <text>
                    render: function (data, type, row) {
                        return (data == true)
                            ? '<i class="fa fa-check true-icon" nop-value="true"></i>'
                            : '<i class="fa fa-close false-icon" nop-value="false"></i>';
                    },
                    </text>
                    break;
                case RenderCustom custom:
                    <text>
                    render: function (data, type, row, meta) {
                        return @(custom.FunctionName)(data, type, row, meta);
                    },
                    </text>
                    break;
                case RenderChildCaret caret:
                    <text>
                    render: function (data, type, row, meta) {
                        return '<i class="fa fa-caret-right" aria-hidden="true"></i>';
                    },
                    </text>
                    break;
            }
            data: '@column.Data'            
        }
        @if (i != Model.ColumnCollection.Count - 1) {<text>,</text>}
        </text>
    }
]
```

### `_GridLocalization.cshtml`

```html
{
    "emptyTable":     "@JavaScriptEncoder.Default.Encode("No data available in table")",
    "info":           "@JavaScriptEncoder.Default.Encode("_START_-_END_ of _TOTAL_ items")",
    "infoEmpty":      "@JavaScriptEncoder.Default.Encode("No records")",
    "infoFiltered":   "@JavaScriptEncoder.Default.Encode("(filtered from _MAX_ total entries)")",
    "thousands":      "@JavaScriptEncoder.Default.Encode(",")",
    "lengthMenu":     "@JavaScriptEncoder.Default.Encode("Show _MENU_ items")",
    "loadingRecords": "@JavaScriptEncoder.Default.Encode("Loading...")",
    "processing":     "@JavaScriptEncoder.Default.Encode("<![CDATA[<i class='fa fa-refresh fa-spin'></i>]]>")",
    "search":         "@JavaScriptEncoder.Default.Encode("Search:")",
    "zeroRecords":    "@JavaScriptEncoder.Default.Encode("No matching records found")",
    "paginate": {
        "first":"<i class='k-icon k-i-seek-w'></i>",
        "last":"<i class='k-icon k-i-seek-e'></i>",
        "next":"<i class='k-icon k-i-arrow-e'></i>",
        "previous":"<i class='k-icon k-i-arrow-w'></i>"
    },
    "aria": {
        "sortAscending":  "@JavaScriptEncoder.Default.Encode(": activate to sort column ascending")",
        "sortDescending": "@JavaScriptEncoder.Default.Encode(": activate to sort column descending")"
    }
}
```

## Step Four - Lets Create the Framework around Fetching the Data

Lets start from the Repository Level. We assume we have a list of players that we need to fetch for. Ignoring how to actually set all of that up (for more information check out [Repository Patterns With Unit Of Work](../efcore/repository-unit-of-work.md)) then our repository method might look something like this:

```c#
public async Task<IEnumerable<PlayerModel>> GetPlayers(PlayerAdminSearchModel request)
{
    return await Context.Player
        .ConditionalWhere(() => request.PlayerName.IsNotNull(), x => x.Name == request.PlayerName)
        .OrderByDescending(x => x.Id)
        .Select(x => _playerToModelMapper.Map(x))
        .ToListAsync();  
}
```

The `ConditionalWhere` can be found in [My Personal Extension Methods - Conditional Where Extension](../extension-methods.md) 

In our Service we would fetch data from the repository using our `UnitOfWorkManager` like so:

```c#
  public async Task<IEnumerable<PlayerModel>> GetPlayerAsync(PlayerAdminSearchModel request)
  {
      return await _unitOfWorkManager.ExecuteSingleAsync
      <IPlayerRepository, IEnumerable<PlayerModel>>
      (u => u.GetPlayers(request));
  }
```

We now create a generic method to map the data accordingly:

```c#
public class BaseAdminTableHandler<TModel, TSearchModel> where TModel : IModel where TSearchModel : BaseSearchModel
{
	public async Task<PagedList<TModel>> SearchPaginated<TService>(TSearchModel searchModel, TService service, Func<TService, 	Task<IEnumerable<TModel>>> runQuery) where TService : IAdminService
    {
        var builder = BasePagedListBuilder<TModel>.Create(
            query: await runQuery(service),
            pageIndex: searchModel.Page - 1,
            pageSize: searchModel.PageSize,
            getOnlyTotalCount: false
        );
        return new PagedList<TModel>(builder.Query, builder.PageIndex, builder.PageSize, builder.GetOnlyTotalCount);
    }
}
```

With Paged List:

```c#
/// <summary>
/// Paged list
/// </summary>
/// <typeparam name="T">T</typeparam>
[Serializable]
public class PagedList<T> : List<T>, IPagedList<T>
{
    /// <summary>
    /// Ctor
    /// </summary>
    /// <param name="source">source</param>
    /// <param name="pageIndex">Page index</param>
    /// <param name="pageSize">Page size</param>
    /// <param name="getOnlyTotalCount">A value in indicating whether you want to load only total number of records. Set to "true" if you don't want to load data from database</param>
    public PagedList(IEnumerable<T> source, int pageIndex, int pageSize, bool getOnlyTotalCount = false)
    {
        var total = source.Count();
        TotalCount = total;
        TotalPages = total / pageSize;

        if (total % pageSize > 0)
            TotalPages++;

        PageSize = pageSize;
        PageIndex = pageIndex;
        if (getOnlyTotalCount)
            return;
        AddRange(source.Skip(pageIndex * pageSize).Take(pageSize).ToList());
    }

    /// <summary>
    /// Ctor
    /// </summary>
    /// <param name="source">source</param>
    /// <param name="pageIndex">Page index</param>
    /// <param name="pageSize">Page size</param>
    public PagedList(IList<T> source, int pageIndex, int pageSize)
    {
        TotalCount = source.Count;
        TotalPages = TotalCount / pageSize;

        if (TotalCount % pageSize > 0)
            TotalPages++;

        PageSize = pageSize;
        PageIndex = pageIndex;
        AddRange(source.Skip(pageIndex * pageSize).Take(pageSize).ToList());
    }

    /// <summary>
    /// Ctor
    /// </summary>
    /// <param name="source">source</param>
    /// <param name="pageIndex">Page index</param>
    /// <param name="pageSize">Page size</param>
    /// <param name="totalCount">Total count</param>
    public PagedList(IEnumerable<T> source, int pageIndex, int pageSize, int totalCount)
    {
        TotalCount = totalCount;
        TotalPages = TotalCount / pageSize;

        if (TotalCount % pageSize > 0)
            TotalPages++;

        PageSize = pageSize;
        PageIndex = pageIndex;
        AddRange(source);
    }

    /// <summary>
    /// Page index
    /// </summary>
    public int PageIndex { get; }

    /// <summary>
    /// Page size
    /// </summary>
    public int PageSize { get; }

    /// <summary>
    /// Total count
    /// </summary>
    public int TotalCount { get; }

    /// <summary>
    /// Total pages
    /// </summary>
    public int TotalPages { get; }

    /// <summary>
    /// Has previous page
    /// </summary>
    public bool HasPreviousPage => PageIndex > 0;

    /// <summary>
    /// Has next page
    /// </summary>
    public bool HasNextPage => PageIndex + 1 < TotalPages;
}
```

And `BasePagedListBuilder` :

```c#
public class BasePagedListBuilder<T> where T : IModel
 {
     public IEnumerable<T> Query { get; set; }
     public int PageIndex { get; set; }
     public int PageSize { get; set; }
     public bool GetOnlyTotalCount { get; set; }

     public static BasePagedListBuilder<T> Create(IEnumerable<T> query, int pageIndex, int pageSize, bool getOnlyTotalCount)
     {
         return new BasePagedListBuilder<T>
         {
             Query = query,
             GetOnlyTotalCount = getOnlyTotalCount,
             PageIndex = pageIndex,
             PageSize = pageSize
         };
     }
 }
```

**Note:** `IModel`, and other Interfaces defined above are simply definition types.

We now create a method to search for the players

```c#
public class PlayerAdminHandler : BaseAdminTableHandler<PlayerModel, PlayerAdminSearchModel>, IPlayerAdminHandler
{
    private readonly IPlayerAdminService _playerAdminService;

    public PlayerAdminHandler(IPlayerAdminService playerAdminService)
    {
        _playerAdminService = playerAdminService;
    }

    public async Task<PagedList<PlayerModel>> SearchPlayers(PlayerAdminSearchModel searchModel)
    {
        return await SearchPaginated(searchModel, _playerAdminService, u => u.GetPlayerAsync(searchModel));      
    }
}
```

We create a factory to control model creation for the view 

```c#
public async Task<PlayerListModel> MakeListModel(PlayerAdminSearchModel searchModel)
{
    var players = await _playerAdminHandler.SearchPlayers(searchModel);
    var playerItems = players.Select(x => new PlayerListItemModel
                                     {
                                         AccountCreatedOnTimeStamp = x.AccountCreatedOnTimeStamp,
                                         AccountLevel = x.AccountLevel,
                                         HoursPlayed = x.HoursPlayed,
                                         LastLoginTimeStamp = x.LastLoginTimeStamp,
                                         MinutesPlayed = x.MinutesPlayed,
                                         Name = x.Name,
                                         PaladinsPlayerId = x.PaladinsPlayerId,
                                         TotalLeaves = x.TotalLeaves,
                                         TotalWins = x.TotalWins
                                     });
    var playerItemsPagedList = new PagedList<PlayerListItemModel>(
        playerItems,
        players.PageIndex,
        players.PageSize,
        players.TotalCount
    );
    return new PlayerListModel().PrepareToGrid(searchModel, players, () => playerItemsPagedList);
}
```

where `PlayerListModel` and `PlayerListItemModel` look like this:

```c#
 public class PlayerListModel : BasePagedListModel<PlayerListItemModel>
 {
 }
 public class PlayerListItemModel
 {
     public int Id { get; set; }
     public string Name { get; set; }
 }

```

The `BasePagedListModel` holds the pagination information and looks like this

```c#
/// <summary>
/// Represents the base paged list model (implementation for DataTables grids)
/// </summary>
public abstract partial class BasePagedListModel<T> : IPagedModel<T>
{
    /// <summary>
    /// Gets or sets data records
    /// </summary>
    public IEnumerable<T> Data { get; set; }

    /// <summary>
    /// Gets or sets draw
    /// </summary>
    public string Draw { get; set; }

    /// <summary>
    /// Gets or sets a number of filtered data records
    /// </summary>
    public int RecordsFiltered { get; set; }

    /// <summary>
    /// Gets or sets a number of total data records
    /// </summary>
    public int RecordsTotal { get; set; }

    public int Total { get; set; }
}
```

In our controller we create an end point responsible for fetching the data 

```c#
 [HttpPost]
 public async Task<JsonResult> Players(PlayerAdminSearchModel searchModel)
 {
     var model = await _playerModelFactory.MakeListModel(searchModel);
     return Json(model);
 }
```



## Step  Five - Implementing the Actual Table (FINALLY OMG)

In our view we have the following:

```c#
 @{ var gridModel = new DataTablesModel
	{
     Name = "player-grid",
     UrlRead = new DataUrl(
         HomeController.PlayerActionName,
         HomeController.ControllerName,
         null),
     Length = Model.PageSize,
     LengthMenu = Model.AvailablePageSizes,
     Filters = new List<FilterParameter>
         {
             new FilterParameter(nameof(Model.PlayerName))
         },
 	};
   gridModel.ColumnCollection.Add(new ColumnProperty(nameof(PlayerListItemModel.Id))
                                  {
                                      Title = "Player Id",
                                      Width = "150",
                                      ClassName = ColumnClassDefaults.CenterAll,
                                  });
   gridModel.ColumnCollection.Add(new ColumnProperty(nameof(PlayerListItemModel.Name))
                                  {
                                      Title = "Player Name",
                                      Width = "200"
                                  });
}
@await Html.PartialAsync("_Table", gridModel)
```

## Conclusion

We now have a way of fetching data and passing it to a datable with minimal code in the View and controller. The solution is adaptable to any situation where `async` repositories fetch the data and ensure that data transition is clean and well architected.

---

## Changelog

- [16-10-2020] - Article On Pagination in MVC using `Datatables.js`  