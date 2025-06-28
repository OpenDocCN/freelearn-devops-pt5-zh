# 第七章：使用 Web API 开发应用程序

Web API 是一项新功能，首次在 Dynamics CRM 2016 中引入。你可以使用 Dynamics 365 Web API，结合不同的编程语言、多个平台和设备。Dynamics 365 中的 Web API 使用 **开放数据协议**（**OData**），也称为 OData 版本 4。由于 Dynamics 365 Web API 基于开放标准构建，因此不需要使用任何程序集。

在 Dynamics 365 中，**组织数据服务**已被弃用，并由 Web API 替代。该 API 的主要目的是提供与组织服务的功能对等，并尽量减少约束。

以下是 Web API 的特点：

+   它实现了 OData 版本 4.0，用于构建和消费基于丰富数据源（如 DOC、HTML 和 PDF）的 RESTful API。

+   它支持多种编程语言，如 .Net、C++、Java、Python、设备和平台。

+   请求和响应采用 JSON 格式

# 开始使用 Dynamics 365 Web API（客户端 JavaScript）

Dynamics 365 Web API 可以通过 JavaScript 调用和访问。你可以将 Web API 与 HTML Web 资源、表单脚本和功能区命令结合使用，对数据执行各种操作。

Web API 与 JavaScript 一起使用非常方便，因为它返回的结果是 JSON 对象，能够轻松地转换为 JavaScript 对象。

在 Dynamics 365 中，Web API 主要用于 HTML Web 资源和单页面应用程序中。

# JavaScript Web 资源

在 JavaScript Web 资源中使用 Web API 的主要好处是，你无需进行身份验证，因为 Web 资源是应用的一部分，只能由已认证用户访问。你可以直接在 JavaScript Web 资源中编写 Web API 操作代码并执行操作。

# 单页面应用程序

单页面应用程序能够发起 Dynamics 365 Web API 调用。它们包含许多运行在浏览器上的 JavaScript 库，这些库通过 **跨源资源共享**（**CORS**）认证 Dynamics 365 API。

在单页面应用程序中使用 JavaScript 时，`adal.js` 库用于允许用户进行身份验证，并通过托管的 Web 应用访问 Dynamics 365。你还必须集成一个包含身份验证令牌的授权头。

在本章后续部分，我们将通过一些示例，使用 Web API 执行基于 Web 资源的 CRUD 操作。

Dynamics 365 Web API 使用 `XMLHttpRequest` 对象来执行操作。

# 在 Dynamics 365 Web API 中使用 XMLHttpRequest

**XMLHttpRequest**（**XHR**）是所有浏览器支持的本地对象，它使得 AJAX 技术得以在网页中动态交互。

我们将查看一个非常简单的示例，该示例使用 Web API 和 `XMLHttpRequest` 对象。

以下是获取所有机会的 Web API 代码：

```
    var req = new XMLHttpRequest();
    req.open("GET", Xrm.Page.context.getClientUrl() +   
    "/api/data/v8.2/opportunities()", true);
    req.setRequestHeader("OData-MaxVersion", "4.0");
    req.setRequestHeader("OData-Version", "4.0");
    req.setRequestHeader("Accept", "application/json");
    req.setRequestHeader("Content-Type",
    "application/json; charset=utf-8");
    req.setRequestHeader("Prefer", "odata.include-annotations="*"");
    req.onreadystatechange = function() {
      if (this.readyState === 4) {
        req.onreadystatechange = null;
        if (this.status === 200) {
          var result = JSON.parse(this.response);
          var opportunityid = result["opportunityid"];
        } else {
        Xrm.Utility.alertDialog(this.statusText);
       }
      }
    };
    req.send();
```

在上述代码中，您会注意到，在初始化一个新的`XMLHttpRequest`对象后，您需要在发送或设置任何属性之前打开它。`open`方法的参数包括 HTTP 请求方法（`GET`、`PUT`、`POST`、`DELETE`等）、URL 以及一个布尔值参数，表示是否异步执行操作。

# Web API URL 和版本

Web API 的 URL 由以下部分组成：

+   **协议**：HTTP 请求中的协议可以是`http://`或`https://`。

+   **基础 URL**：基础 URL 就是当前组织的 URL，可以通过函数`Xrm.Page.context.getClientUrl()`获取。

+   **Web API 路径**：Dynamics 365 中的 Web API 路径是 API/data。

+   **版本**：这是 Dynamics 365 Web API 的版本。最新版本是 9.0。

+   **资源**：资源可以是实体的名称、函数或您想执行的操作。

上述示例中使用的 URL 是`Xrm.Page.context.getClientUrl() + "/api/data/v8.2/opportunities()`.

HTTP 请求还支持各种**HTTP 方法**，具体描述如下：

+   `GET`：用于检索数据；成功调用的状态码是`200 OK`

+   `POST`：用于创建新记录

+   `PATCH`：用于更新或执行对实体记录的`upsert`操作

+   `DELETE`：用于删除记录

+   `PUT`：用于更新实体记录的单个属性

对于 HTTP 请求，还使用了各种 HTTP 头部，具体描述如下：

Dynamics 365 仅支持 JSON 格式。因此，以下头部可与 Dynamics 365 Web API 一起使用。

对于每个请求，必须包括`Accept`头部值`application/json`，即使没有响应，返回的也是正文。如果发生错误，将以 JSON 格式返回。虽然代码没有这个头部也能正常工作，但在请求中使用它是一种最佳实践。

您必须始终包含`OData-Version`和`OData-Max-Version`头部，设置为 4.0 的值。当前的 OData 版本是 4.0，但为了避免未来 OData 版本的不明确性，您应该在请求中包括这些头部。

不包含任何最近更改的属性可能包含缓存数据。因此，为了覆盖浏览器缓存 Web API 请求，必须在请求正文中包含`If-None-Match: null`头部**。**

在对 Dynamics 365 中的 Web API 有了基本了解后，我们将开始使用 Web API 执行各种操作。我们还将学习如何使用 Web API 请求创建、检索、更新和删除实体记录。我们还会查看关联和取消关联请求的示例。

因此，至少应包括以下 HTTP 头部：

+   `Accept: application/json`

+   `OData-MaxVersion: 4.0`

+   `OData-Version: 4.0`

+   `If-None-Match: null`

# 使用 Dynamics 365 Web API 查询数据

在从 Dynamics 365 检索数据时，您可以设置所需数据的各种条件，并且可以应用多种过滤器来检索特定的数据。为此，我们将通过一个示例来展示如何使用 Web API 查询数据：

1.  在此示例中，我们将使用 `$set` 和 `$top` 系统查询选项返回前五个联系人的 `firstname` 属性：

```
   Request: GET [Organization URI] /api/data/v9.0/contacts? 
    $select=firstname&$top=5
    Accept: application/json
    OData-MaxVersion: 4.0
    OData-Version: 4.0
```

前述请求的响应如下：

```
    {
      "@odata.context":  "https://biocondev3.crm8.dynamics.com
      /api/data/v9.0/$metadata#contacts(firstname)",
      "value": [
      {
        "@odata.etag": "W/"684628"",
        "firstname": null,
        "contactid": "ade1d0f5-28ed-e711-a95e-000d3af27347"
      },
      {
        "@odata.etag": "W/"679541"",
        "firstname": null,
        "contactid": "fa420510-85ec-e711-a95f-000d3af27534"
      },
      {
        "@odata.etag": "W/"679547"",
        "firstname": "C",
        "contactid": "94cd4c79-85ec-e711-a95f-000d3af27534"
      },
      {
        "@odata.etag": "W/"679575"",
        "firstname": null,
        "contactid": "e8f5339f-88ec-e711-a95f-000d3af27534"
      },
      {
        "@odata.etag": "W/"681230"",
        "firstname": null,
        "contactid": "fc71e9b9-a2ec-e711-a95f-000d3af27534"
      }
    ]
   }
```

1.  在下一个示例中，我们将展示如何限制每个请求返回的实体记录数。最多可以返回 5,000 条记录。您不能在 Dynamics 365 中检索超过 5,000 条记录：

```
Request: GET [Organization URI]/api/data/v9.0/accounts?$select=nameHTTP/1.1
Accept: application/json
OData-MaxVersion: 4.0
OData-Version: 4.0
Prefer: odata.maxpagesize=3
```

前述请求的响应如下：

```
{
"@odata.context": "https://biocondev3.crm8.dynamics.com/api/data/v9.0/$metadata#contacts(firstname)",
"value": [
{
"@odata.etag": "W/"684628"",
"firstname": null,
"contactid": "ade1d0f5-28ed-e711-a95e-000d3af27347"
},
{
"@odata.etag": "W/"679541"",
"firstname": null,
"contactid": "fa420510-85ec-e711-a95f-000d3af27534"
},
{
"@odata.etag": "W/"679547"",
"firstname": "C",
"contactid": "94cd4c79-85ec-e711-a95f-000d3af27534"
}
]
}
```

1.  现在，我们将看到如何使用 Dynamics 365 Web API 应用系统查询选项来查询数据。第一个查询选项附加在 [`?`] 后面，后续每个选项通过 [`&`] 分隔。查询选项区分大小写。

在下一个查询中，我们将选择年龄小于 `50` 的联系人的 `firstname` 和 `lastname`：

```
GET [Organization URI]/api/data/v9.0/contacts?$select=firstname,lastname&$top=3&$filter=age lt 50
```

Dynamics 365 Web API 中使用的标准筛选运算符如下所示：

| **运算符** | **描述** | **示例** |
| --- | --- | --- |
| **比较运算符** | - | - |
| `Eq` | 等于 | `$filter=age eq 50` |
| `Ne` | 不等于 | `$filter=age ne 50` |
| `Gt` | 大于 | `$filter=age gt 50` |
| `Ge` | 大于或等于 | `$filter=age ge 50` |
| `Lt` | 小于 | `$filter=age lt 50` |
| `Le` | 小于或等于 | `$filter=age le 50` |

逻辑运算符如下所示：

| **运算符** | **描述** | **示例** |
| --- | --- | --- |
| **逻辑运算符** | - | - |
| `And` | 逻辑与 | `$filter=age lt 50 and age gt 20` |
| `Or` | 逻辑或 | `$filter=contains(firstname,'(sample)') or contains(firstname,'test')` |
| `Not` | 逻辑非 | `$filter=not contains(firstname,'sample')` |

分组运算符如下所示：

| **分组运算符** | - | - |
| --- | --- | --- |
| `( )` | 优先级分组 | `(contains(firstname,'sample') or contains(firstname,'test')) and age gt 50` |

# 标准查询选项

Web API 支持的 OData 字符串查询函数如下所示：

| **函数** | **示例** |
| --- | --- |
| `Contains` | `$filter=contains(firstname,'(sample)')` |
| `Endswith` | `$filter=endswith(firstname,'Inc.')` |
| `startswith` | `$filter=startswith(firstname,'a')` |

**排序查询**：

您可以按升序或降序排序记录。以下示例展示了如何排序记录：

```
GET [Organization URI]/api/data/v9.0/contacts?$select=firstname,age,&$orderby=age asc,firstname desc&$filter=age ne null
```

现在，学习了如何使用 Dynamics 365 Web API 查询数据后，我们将讨论 CRUD 操作。我们将通过一些执行这些操作的示例来进行演示。

# 使用 Dynamics 365 Web API 进行 CRUD 操作

我们将展示一些创建、更新、检索和删除实体的基本示例。

**创建实体**

以下是一个 Web API 的代码示例，用于为 `entity` 账户创建新记录。首先，我们将创建一个 JSON 对象并设置所需的属性：

```
//Creates a JSON object for an Entity to be created.
var entity = {};
//Sets the properties for entity.
entity.accountnumber = "AC1009348YU";
entity.name = "Shree Technosoft";
entity.telephone1 = "9978759612";
var req = new XMLHttpRequest();
req.open("POST", Xrm.Page.context.getClientUrl() + "/api/data/v8.2/accounts", true);
req.setRequestHeader("OData-MaxVersion", "4.0");
req.setRequestHeader("OData-Version", "4.0");
req.setRequestHeader("Accept", "application/json");
req.setRequestHeader("Content-Type", "application/json; charset=utf-8");
req.onreadystatechange = function() {
  if (this.readyState === 4) {
    req.onreadystatechange = null;
    if (this.status === 204) {
      var uri = this.getResponseHeader("OData-EntityId");
      var regExp = /(([^)]+))/;
      var matches = regExp.exec(uri);
      var newEntityId = matches[1];
    } else {
      Xrm.Utility.alertDialog(this.statusText);
     }
   }
  };
  req.send(JSON.stringify(entity));
```

**获取实体记录列表**

接下来，我们将通过 Web API 请求从 Dynamics 365 中检索账户列表。以下是用于检索实体列表的代码：

```
var req = new XMLHttpRequest();
req.open("GET", Xrm.Page.context.getClientUrl() + "/api/data/v8.2/accounts?$select=accountid,accountnumber,accountratingcode,telephone1", true);
req.setRequestHeader("OData-MaxVersion", "4.0");
req.setRequestHeader("OData-Version", "4.0");
req.setRequestHeader("Accept", "application/json");
req.setRequestHeader("Content-Type", "application/json; charset=utf-8");
req.onreadystatechange = function() {
  if (this.readyState === 4) {
    req.onreadystatechange = null;
    if (this.status === 200) {
      var results = JSON.parse(this.response);
      for (var i = 0; i < results.value.length; i++) {
        var accountid = results.value[i]["accountid"];
        var accountnumber = results.value[i]["accountnumber"];
        var accountratingcode = results.value[i]
        ["accountratingcode"];
        var telephone1 = results.value[i]["telephone1"];
     }
   } else {
       Xrm.Utility.alertDialog(this.statusText);
     }
   }
  };
  req.send();
  The response for the above code will be as below:
  {
    "@odata.context":    "https://demoorg3.crm8.dynamics.com/api/
    data/v8.2/$metadata#accounts
    (accountid,accountnumber,accountratingcode,telephone1)",
    "value": [
    {
      "@odata.etag": "W/"671215"",
      "accountid": "9252cd68-e9e3-e711-a95e-000d3af27163",
      "accountnumber": "DR567821X",
      "accountratingcode": 1,
      "telephone1": null
   },
   {
     "@odata.etag": "W/"677922"",
     "accountid": "e771ecea-edea-e711-a95f-000d3af27534",
     "accountnumber": "DR9888900RR",
     "accountratingcode": 1,
     "telephone1": null
   },
   {
     "@odata.etag": "W/"707246"",
     "accountid": "2de6ad4f-0ef1-e711-a95e-000d3af278ae",
     "accountnumber": "AC1009348YU",
     "accountratingcode": 1,
     "telephone1": "9978759612"
   },
   {
     "@odata.etag": "W/"684666"",
     "accountid": "493d50f5-28ed-e711-a95e-000d3af27fd9",
     "accountnumber": "DR7845699AS",
     "accountratingcode": 1,
     "telephone1": null
   }
  ]
 }
```

这些检索到的对象可以很容易地转换为 JavaScript 对象。这些对象可以分配给 JavaScript 数组，并可用于后续操作。

**更新实体记录**

现在，我们将更新一个`entity`记录。更新记录时，你需要传递你想要更新的记录的 GUID，以及包含要更新字段的 JSON 对象。在这里，我们将更新一个账户记录的`email`和`city`：

```
var entity = {};
entity.emailaddress1 = "contact@shreetech.in";
entity.address1_city = "Mumbai";
var req = new XMLHttpRequest();
req.open("PATCH", Xrm.Page.context.getClientUrl() + "/api/data/v8.2/accounts(2de6ad4f-0ef1-e711-a95e-000d3af278ae)", true);
req.setRequestHeader("OData-MaxVersion", "4.0");
req.setRequestHeader("OData-Version", "4.0");
req.setRequestHeader("Accept", "application/json");
req.setRequestHeader("Content-Type", "application/json; charset=utf-8");
req.onreadystatechange = function() {
  if (this.readyState === 4) {
    req.onreadystatechange = null;
    if (this.status === 204) {
      //Success - No Return Data - Do Something
    } else {
      Xrm.Utility.alertDialog(this.statusText);
    }
   }
 };
 req.send(JSON.stringify(entity));
```

**删除实体记录**

删除`entity`记录时，你需要传递要删除记录的 GUID。以下是从 Dynamics 365 删除账户`entity`记录的代码：

```
var req = new XMLHttpRequest();
req.open("DELETE", Xrm.Page.context.getClientUrl() + "/api/data/v8.2/accounts(2de6ad4f-0ef1-e711-a95e-000d3af278ae)", true);
req.setRequestHeader("Accept", "application/json");
req.setRequestHeader("Content-Type", "application/json; charset=utf-8");
req.setRequestHeader("OData-MaxVersion", "4.0");
req.setRequestHeader("OData-Version", "4.0");
req.onreadystatechange = function () {
  if (this.readyState === 4) {
    req.onreadystatechange = null;
    if (this.status === 204 || this.status === 1223) {
      //Success - No Return Data - Do Something
    } else {
      Xrm.Utility.alertDialog(this.statusText);
    }
  }
};
req.send();
```

# 在 Dynamics 365 Web API 中进行模拟操作

当你想要代表另一个用户执行业务逻辑时，你需要使用模拟。有时候，某些流程或业务逻辑需要代表特定的 Dynamics 365 用户来执行，并且该用户拥有适当的安全角色；对于这种需求，模拟非常有用。

在代码中，当你需要代表其他用户创建记录时，你将使用此功能。为此，你需要两个用户账户。要使用模拟功能，你需要添加一个名为 `MSCRMCallerID` 的头部，其 GUID 值等于该用户的系统用户 ID，具体如下请求所示。

请参考以下代码：

```
Request
POST [Organization URI]/api/data/v9.0/contacts HTTP/1.1
MSCRMCallerID: 9ba9f608-514e-49fd-81cd-84537a31f68e
Accept: application/json
Content-Type: application/json; charset=utf-8
OData-MaxVersion: 4.0
OData-Version: 4.0
{"name":"Contact created using impersonation"}
```

# 使用 Web API 获取元数据

Microsoft Dynamics 365 是一个元数据驱动的应用程序，在某些场景和特定需求中，你需要查询元数据。Dynamics 365 Web API 支持查询元数据。我们将使用`EntityDefinitions`实体来获取`contact`实体的元数据。

以下是查询元数据的请求：

```
GET [OrgURL]/api/data/v8.2/EntityDefinitions(LogicalName='contact')/Attributes"
Following is the Sample code for querying metadata for Contact Entity:
var req = new XMLHttpRequest();
//Opens request
req.open("GET", window.parent.Xrm.Page.context.getClientUrl() + "/api/data/v8.2/EntityDefinitions(LogicalName='contact')/Attributes", true);
//Sets request Headers
req.setRequestHeader("OData-MaxVersion", "4.0");
req.setRequestHeader("OData-Version", "4.0");
req.setRequestHeader("Accept", "application/json");
req.setRequestHeader("Content-Type", "application/json; charset=utf-8");
req.setRequestHeader("Prefer", "odata.include-annotations="*"");
//Function to detect ready state change
req.onreadystatechange = function () {
  if (this.readyState === 4) {
    req.onreadystatechange = null;
    //Check if request completed successfully
    if (this.status === 200) {
      var result = JSON.parse(this.response);
      var index = 1;
      for (var i = 0; i < result.value.length; i++) {
        //Gets schemaname from response.
        var schemaName = result.value[i].SchemaName;
      }
    }
    else {
      Xrm.Utility.alertDialog(this.statusText);
    }
   }
 };
 req.send();
});
```

# Dynamics 365 Web API 在 9.0 版本中的更新

现在，在 Dynamics 365 最新发布的 9.0 版本中，对于查询数据和执行各种操作，使用 Dynamics 365 Web API 进行了许多显著的变化。随着新版本的发布，我们将不再需要为使用 Web API 创建任何请求，而是直接使用内置的预定义函数来使用 Web API。

在新版本的 Dynamics 365 中使用 Web API 变得非常简单和方便。现在，Dynamics 365 引入了新的库`Xrm.WebApi`，并通过 Web API 执行 CRUD 操作。该库提供了执行操作的函数，接下来我们将在示例和代码中展示这些操作。

**创建实体记录**

这是使用`Xrm.WebApi.createRecord()`函数创建新联系人的示例代码：

```
function createContact() {
  // contact data object
  var entity = {};
  entity.firstname = "Mark";
  entity.lastname = "Andrew";
  entity.telephone1 = "7845687845";
  entity.address1_city = "California";
  // create record
  Xrm.WebApi.createRecord("contact", entity).then(
  function success(result) {
    console.log("Contact Created Successfully !!");
  },
  function (error) {
    console.log(error.message);
  }
 );
}
```

**更新实体记录**

在以下示例中，我们将通过添加一个新的`email`属性来更新联系人记录：

```
function updateContact() {
  // contact data object
  var entity = {};
  entity.emailaddress1 = "mark@gmail.com";
  //get contact id
  var contactId = "DC1F674E-55F1-E711-A95F-000D3AF27163"
  // update contact record
  Xrm.WebApi.updateRecord("contact", contactId, entity).then(
  function success(result) {
    console.log("Contact Record Updated");
  },
  function (error) {
    console.log(error.message);
  }
 );
}
```

**删除实体记录**

以下是删除联系人记录的示例代码：

```
function deleteContact() {
  var contactId = "DC1F674E-55F1-E711-A95F-000D3AF27163";
  Xrm.WebApi.deleteRecord("contact", contactId).then(
  function success(result) {
    console.log("Contact deleted");
  },
  function (error) {
    console.log(error.message);
  }
 );
}
```

**检索记录**

在新版本中，你可以直接使用 fetch XML 通过 Web API 从 Dynamics 365 检索多个记录。在下面的示例中，我们将使用 fetch XML 查询来获取/检索某个选定账户的联系人。以下是从 Dynamics 365 检索记录的示例代码：

```
function retrieveMultipleContacts(executioncontext) {
  var formContext = executioncontext.getFormContext();
  var contactId = formContext.data.entity.getId();
  var fetchContacts = "<fetch version='1.0' output-format='xml-  
  platform' mapping='logical' distinct='false'>" +
  "<entity name='contact' >" +
  "<attribute name='fullname' />" +
  "<attribute name='telephone1' />" +
  "<attribute name='contactid' />" +
  "<order attribute='fullname' descending='false' />" +
  "<filter type='and'>" +
  "<condition attribute='parentcustomerid' operator='eq'   
  uitype='account' value ='" + accountId + "' />" +
  "</filter>" +
  "</entity >" +
  "</fetch > ";
  Xrm.WebApi.retrieveMultipleRecords("contact", "fetchXml= " +    
  fetchContacts).then(
    function success(result) {
    for (var i = 0; i < result.entities.length; i++) {
      console.log(result.entities[i]);
    }
  },
   function (error) {
     console.log(error.message);
    });
  }
```

# 总结

在本章中，我们学习了如何在 Dynamics 365 Web API 中处理单页应用程序、XMLHttpRequest、Web API URL 和版本，以及标准查询选项，还学习了如何使用 Dynamics 365 Web API 进行 CRUD 操作。在下一章中，我们将介绍 Azure 与 Dynamics 365 的集成，如何配置 Azure 集成与 Dynamics 365，并编写支持 Azure 的插件和不同的监听应用程序。
