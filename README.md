# Dynamically Filter a Lookup field with Client Script

Lookup fields are great for linking records together inside your CRM. However, one of the most common requests I receive is how to filter the lookup results quickly without clicking on the lookup search.

This article will go through how you can filter lookups both statically through the Layout Builder and dynamically using client scripts.

## Static Lookup Filtering

If you are not aware, you can filter lookup fields by a static value in the layout builder. This feature is great if you want to display records for a lookup based on a specific value.

For example, let's say you have a lookup field called "Supplier Account" assigned to a deal. This allows you to select the account supplying the goods and services for this deal, and you only want to display "Supplier" account types in the lookup.

To do this, you can go into the Layout Builder, open up the Lookup Properties, and select 'Filter lookup records.' Here, you can configure the criteria for that lookup. In this example, we only want accounts where the account type is 'Supplier.'

![Screenshot of the Lookup Properties](https://static.wixstatic.com/media/c8c3af_b7403705832c40899b8a44d8693b209f~mv2.png/v1/fill/w_740,h_417,al_c,q_85,usm_0.66_1.00_0.01,enc_auto/c8c3af_b7403705832c40899b8a44d8693b209f~mv2.png)

Now, when we use this lookup inside the record, the results are filtered for us and our selection options are reduced.

Let’s take this a bit further and say we want to assign a "Supplier Contact" that we are working with for this deal. Zoho CRM doesn't currently provide the ability to filter the "Supplier Contact" field to only show contacts related to the chosen "Supplier Account."

This is where client scripts come in handy. With client scripts, you can dynamically filter a lookup field based on another field in the record. For instance, you can filter the "Supplier Contact" field to only show contacts associated with the selected "Supplier Account."

## What are client scripts?

Client scripts are custom scripts that run on the client side within the Zoho CRM interface to extend or enhance the functionality of the CRM system. They execute within your browser as apposed to when you save the record.

They differ from custom functions, which are server-side and only trigger via a workflow when the record is saved. Think of client scripts as in-browser workflows that can assist you while you are creating or editing a record by filtering lookups, pre-filling fields with values, or validating data. They can trigger on specific pages such as the Create, Clone, Edit, and Details (View) pages, and on specific events, such as page events like load, change, and save, or field events such as change and type.

Another difference with client scripts is that they are not written in Zoho Deluge but in JavaScript, as they run in the browser.

## Creating client scripts

To configure client scripts in your CRM, you need to go to Settings > Developer Hub > Client Scripts.

When planning your client scripts, ensure that you create a script for each of the possible pages and events required.

In our example, we want the "Supplier Contact" field to only show contacts associated with the selected "Supplier Account." This means we will need to create client scripts to trigger on the following pages and events:

1. Create Page > Field Event (Supplier Account) > onChange
2. Clone Page > Field Event (Supplier Account) > onChange
3. Clone Page > Page Event > onLoad
4. Edit Page > Field Event (Supplier Account) > onChange
5. Edit Page > Page Event > onLoad
6. Details Page > Page Event > onLoad
7. Details Page > Field Event (Supplier Account) > onBeforeUpdate

In our example, the same script will be used for 6 of the 7 events, so we can copy and paste it, or you can create functions that you can call in the [Static Resources](https://www.zoho.com/crm/developer/docs/client-script/static-resources.html) section, which we will cover in another article.

To start, click the 'New Script' button, then configure the page and event triggers for your client script. Write your script or copy the one below as a template.

Example code all events above (1 - 6)
Filter Supplier Contact
``` js
//Get the Supplier Account & Supplier Contact Field Objects
const supplierAccountField = ZDK.Page.getField('Supplier_Account');
const supplierContactField = ZDK.Page.getField('Supplier_Contact');
//Check the Supplier Account Field is not empty
if (supplierAccountField.getValue()) {
    //Get the Name of the Supplier Account
    const supplierAccountName = supplierAccountField.getValue().name;
    //Add criteria to filter the lookup field
    supplierContactField.setCriteria("(Account_Name:equals:" + supplierAccountName + ")", {filterOnSearch: true});
}
```

The last event on the list (Details Page > Field Event > onBeforeUpdate) needs to be a little different as it occurs during the in-line edit of a lookup. We need to get the value of the "Supplier Account" field during the save and apply the lookup filtering to the "Supplier Contact" field. During this type of event, the client script passes in a 'value' argument that we need to use.

To start, click the 'New Script' button, then configure the page and event triggers for your client script. Write your script or copy the one below as a template.

Example code for event 7: Details Page > Field Event (Supplier Account) > onBeforeUpdate
Filter Supplier Contact (onBeforeUpdate)
```js
//Get the New Supplier Account Selected before save
const supplierAccountName = value;
//Get the Supplier Contact Field Object
const supplierContactField = ZDK.Page.getField('Supplier_Contact');
//Check the Supplier Account Field is not empty
if (supplierAccountName) {
    //Add criteria to filter the lookup field
    supplierContactField.setCriteria("(Account_Name:equals:" + supplierAccountName + ")", {filterOnSearch: true});
}
```

## Client Script Limitations

Currently, there are some limitations to client scripts. I've listed a few that I have encountered:

1. They are not supported on the 'Quick Create' Menu.
2. If you have a script attached to the 'stage' field on the Deals module, they do not trigger when a user changes the stage via the progress bar.
3. Multi-select picklist fields do not trigger on change.

I hope this article has helped you configure some client scripts in your CRM. Filtering lookup fields is just one of the many things you can do with client scripts. Check out the documentation below for more information.

Need Help? Contact us!

Resources

https://www.zoho.com/crm/developer/docs/client-script/overview.html
