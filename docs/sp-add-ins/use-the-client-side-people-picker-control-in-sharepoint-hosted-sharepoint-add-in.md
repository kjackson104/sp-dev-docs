---
title: Use the client-side People Picker control in SharePoint-hosted SharePoint Add-ins
description: Use the client-side People Picker control to quickly search for and select valid user accounts for people, groups, and claims in your organization.
ms.date: 12/20/2017
ms.prod: sharepoint
ms.localizationpriority: high
---

# Use the client-side People Picker control in SharePoint-hosted SharePoint Add-ins

> [!IMPORTANT] 
> This topic assumes that you know how to create a SharePoint-hosted SharePoint Add-in. To learn how to create one, see [Get started creating SharePoint-hosted SharePoint Add-ins](get-started-creating-sharepoint-hosted-sharepoint-add-ins.md).
 
<a name="bk_whatIs"> </a>

## What is the client-side People Picker control in SharePoint?

The client-side People Picker control lets users quickly search for and select valid user accounts for people, groups, and claims in their organization. The picker is an HTML and JavaScript control that provides cross-browser support. 

Adding the picker to your add-in is easy: 

1. In your markup, add a container element for the control and references for the control and its dependencies. 

2. In your script, call the **SPClientPeoplePicker_InitStandaloneControlWrapper** global function to render and initialize the picker.

The picker is represented by the **SPClientPeoplePicker** object, which provides methods that other client-side controls can use to get information from the picker or to perform other operations. You can use **SPClientPeoplePicker** properties to configure the picker with control-specific settings, such as allowing multiple users or specifying caching options. 

The picker also uses web application configuration settings such as Active Directory Domain Services parameters or targeted forests. Web application configuration settings are picked up automatically (from the **SPWebApplication.PeoplePickerSettings** property).
 
The picker has the following components:

- An input text box to enter names for users, groups, and claims.
- A span control that shows the names of resolved users, groups, and claims.
- A hidden **div** element that autofills a drop-down box with matching query results.
- An autofill control.
    
> [!NOTE] 
> The picker and its functionality are defined in the **clientforms.js**, **clientpeoplepicker.js**, and **autofill.js** script files, which are located in the %ProgramFiles%\Common Files\Microsoft Shared\web server extensions\15\TEMPLATE\LAYOUTS folder on the server.
 
<a name="bk_prereqs"> </a>

## Prerequisites for setting up your development environment to use the client-side People Picker control in a SharePoint Add-in

This article assumes that you create the SharePoint Add-in by using Napa on an Office 365 developer site. If you're using this development environment, you've already met the prerequisites.
 
> [!NOTE] 
> To find out how to sign up for a developer site and start using Napa, see [Set up a development environment for SharePoint Add-ins on Office 365](set-up-a-development-environment-for-sharepoint-add-ins-on-office-365.md).
 
If you're not using Napa on a developer site, you need the following:

- SharePoint with at least one target user
- Visual Studio 2012 or Visual Studio 2013
- Office Developer Tools for Visual Studio 2013
    
> [!NOTE] 
> For guidance about how to set up a development environment that fits your needs, see [SharePoint Add-ins](sharepoint-add-ins.md#two-types-of-sharepoint-add-ins-sharepoint-hosted-and-provider-hosted).
 
The following sections describe the high-level steps for adding the picker to your add-in and then getting its user information from another client-side control. For the corresponding code, see [Code example: Using the client-side People Picker](#bk_example).

You can use the client-side People Picker control in SharePoint-hosted SharePoint Add-ins, but not in provider-hosted add-ins. 

<a name="bk_steps"> </a>

## Use the client-side People Picker control in a SharePoint-hosted add-in

### In your page markup

1. Add references to the client-side People Picker control's script dependencies.

2. Add the HTML tags for the page UI. The add-in in this example defines two **div** elements: one for the picker to render in and one for the UI: a button that invokes script to query the picker and the elements that display user information.
    
### In your script

1. Create a JSON dictionary to use as a schema that stores picker-specific properties, such as **AllowMultipleValues** and **MaximumEntitySuggestions**.

2. Call the **SPClientPeoplePicker_InitStandaloneControlWrapper** global function.

3. Get the picker object from the page.

4. Query the picker. The add-in in this example calls the **GetAllUserInfo** method to get all user information for the resolved users, and the **GetAllUserKeys** method to just get the keys for the resolved users.

5. Get the user ID by using the JavaScript object model. The user ID isn't included in the information that's returned by the picker, so the add-in calls the **SP.Web.ensureUser** method and gets the ID from the returned **SP.User** object.

Rendering, initializing, and other functionality for the picker are handled by the server, including searching and resolving user input against the SharePoint authentication providers.
 
<a name="bk_example"> </a>

## Code example: Using the client-side People Picker in a SharePoint-hosted add-in

The following HTML and JavaScript code examples add a client-side People Picker control to a SharePoint-hosted add-in.

The first example shows the page markup for the **PlaceHolderMain asp:Content** tags in the Default.aspx page. This code references the picker's script dependencies, gives a unique ID to the DOM element where the picker renders, and defines the add-in's UI.

```HTML
<asp:Content ContentPlaceHolderId="PlaceHolderMain" runat="server">
    <SharePoint:ScriptLink name="clienttemplates.js" runat="server" LoadAfterUI="true" Localizable="false" />
    <SharePoint:ScriptLink name="clientforms.js" runat="server" LoadAfterUI="true" Localizable="false" />
    <SharePoint:ScriptLink name="clientpeoplepicker.js" runat="server" LoadAfterUI="true" Localizable="false" />
    <SharePoint:ScriptLink name="autofill.js" runat="server" LoadAfterUI="true" Localizable="false" />
    <SharePoint:ScriptLink name="sp.js" runat="server" LoadAfterUI="true" Localizable="false" />
    <SharePoint:ScriptLink name="sp.runtime.js" runat="server" LoadAfterUI="true" Localizable="false" />
    <SharePoint:ScriptLink name="sp.core.js" runat="server" LoadAfterUI="true" Localizable="false" />
    <div id="peoplePickerDiv"></div>
    <div>
        <br/>
        <input type="button" value="Get User Info" onclick="getUserInfo()"></input>
        <br/>
        <h1>User info:</h1>
        <p id="resolvedUsers"></p>
        <h1>User keys:</h1>
        <p id="userKeys"></p> 
        <h1>User ID:</h1>
        <p id="userId"></p>
    </div>
</asp:Content>
```

<br/>

> [!NOTE] 
> Depending on your environment, you might not have to explicitly reference all of these dependencies.
 
<br/>

The following example shows the **script from the App.js file**. This script initializes and renders the picker, queries it for user information, and then gets the user ID by using the JavaScript object model.

```javascript

// Run your custom code when the DOM is ready.
$(document).ready(function () {

    // Specify the unique ID of the DOM element where the
    // picker will render.
    initializePeoplePicker('peoplePickerDiv');
});

// Render and initialize the client-side People Picker.
function initializePeoplePicker(peoplePickerElementId) {

    // Create a schema to store picker properties, and set the properties.
    var schema = {};
    schema['PrincipalAccountType'] = 'User,DL,SecGroup,SPGroup';
    schema['SearchPrincipalSource'] = 15;
    schema['ResolvePrincipalSource'] = 15;
    schema['AllowMultipleValues'] = true;
    schema['MaximumEntitySuggestions'] = 50;
    schema['Width'] = '280px';

    // Render and initialize the picker. 
    // Pass the ID of the DOM element that contains the picker, an array of initial
    // PickerEntity objects to set the picker value, and a schema that defines
    // picker properties.
    this.SPClientPeoplePicker_InitStandaloneControlWrapper(peoplePickerElementId, null, schema);
}

// Query the picker for user information.
function getUserInfo() {

    // Get the people picker object from the page.
    var peoplePicker = this.SPClientPeoplePicker.SPClientPeoplePickerDict.peoplePickerDiv_TopSpan;

    // Get information about all users.
    var users = peoplePicker.GetAllUserInfo();
    var userInfo = '';
    for (var i = 0; i < users.length; i++) {
        var user = users[i];
        for (var userProperty in user) { 
            userInfo += userProperty + ':  ' + user[userProperty] + '<br>';
        }
    }
    $('#resolvedUsers').html(userInfo);

    // Get user keys.
    var keys = peoplePicker.GetAllUserKeys();
    $('#userKeys').html(keys);

    // Get the first user's ID by using the login name.
    getUserId(users[0].Key);
}

// Get the user ID.
function getUserId(loginName) {
    var context = new SP.ClientContext.get_current();
    this.user = context.get_web().ensureUser(loginName);
    context.load(this.user);
    context.executeQueryAsync(
         Function.createDelegate(null, ensureUserSuccess), 
         Function.createDelegate(null, onFail)
    );
}

function ensureUserSuccess() {
    $('#userId').html(this.user.get_id());
}

function onFail(sender, args) {
    alert('Query failed. Error: ' + args.get_message());
}
```

<br/>

## See also
<a name="bk_addresources"> </a>

-  [Create UX components in SharePoint](create-ux-components-in-sharepoint.md)
-  [People Picker and claims providers overview (SharePoint)](https://technet.microsoft.com/library/gg602078.aspx)
-  [Configure People Picker in SharePoint](https://technet.microsoft.com/library/gg602075.aspx)
    
 

