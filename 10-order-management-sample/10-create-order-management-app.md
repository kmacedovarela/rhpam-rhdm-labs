Order Management Process
========================

This is a Process Management lab in which will implement an Order Management process. The process will use BPMN2 constructs like **Swimlanes**, **User Tasks**, **Gateways**, combined with decision-based routing based on a **DMN Model** (Decision Model & Notation). It also introduces more dynamic concepts of the Red Hat Process Automation Manager process engine, like dynamic assignments of tasks based on process instance data.

## Goals

-   Create an Order Management project in Red Hat Process Automation
    Manager.
-   Define and create the process' domain model using the platform’s
    Data Modeller.
-   Implement an order management process in the process designer
-   Implement decision logic in a DMN model.
-   Create forms with the platform’s Form Modeller.
-   Deploy the project to the platform’s Execution Server.
-   Execute the end-to-end process.

## Pre-reqs

-   Successful completion of the *Environment Setup Lab* or

-   An existing, accessible, DM/PAM 7.3+ environment.

Problem Statement
=================

In this lab we will create an Order Management process that manages the process of ordering a new phone or laptop.

-   Start the process by providing the order information.
-   The supplier sends an offer stating the expected delivery date and its best offer.
-   Depending on the urgency of the urgency and the price, the order can be auto-approved by a DMN decision.
-   If the order is not auto-approved, the manager needs to complete an approval step.

Create a Project
================

To define and deploy a business process, we first need to create a new project in which we can store the BPMN2 model, our domain model and the forms required for user interaction. To create a new project:

1.  Navigate to [Business Central](http://localhost:8080/business-central)
    
2.  Login to the platform with the provided username and password.

3.  Click on **Design** to navigate to the Design perspective.

    ![](images/bc-design-empty.png)

4.  In the Design perspective, create a new project. If your space is empty, this can be done by clicking on the blue **Add Project** button in the center of the page. If you already have projects in your space, you can click on the blue **Add Project** icon at the top right of the page.
    
5.  Give the project the name `order-management`, and the description "Order Management".
    

![](images/create-order-management-project.png)

With the project created, we can now start building our solution.

Solution
========

The Domain Model
----------------

The business process will collect and carry data through the execution of the process. This data is stored in a data model or domain model. In this lab, we collect two types of data:

-   `OrderInfo`: contains information about the order, like the item and the price.
    
- `SupplierInfo`: contains information about the supplier, like the name and the expected delivery date.

  

1. In your project, click on the *Add Asset* button in the middle of the screen.

![](images/add-asset-button.png)

2. In the drop-down menu in the upper-left corner, select `Model`. Click on the *Data Object*
   tile.

![](images/new-data-object.png) 

3. Give the *Data Object* the name `OrderInfo`. Leave the package set to default.

![](images/new-data-object-order-info.png) . Add the following

* Fields to the `OrderInfo` data object:

<table>
<colgroup>
<col style="width: 33%" />
<col style="width: 33%" />
<col style="width: 33%" />
</colgroup>
<tbody>
<tr class="odd">
<td><p>Identifier</p></td>
<td><p>Label</p></td>
<td><p>Type</p></td>
</tr>
<tr class="even">
<td><p>item</p></td>
<td><p>item name</p></td>
<td><p>String</p></td>
</tr>
<tr class="odd">
<td><p>urgency</p></td>
<td><p>urgency</p></td>
<td><p>String</p></td>
</tr>
<tr class="even">
<td><p>targetPrice</p></td>
<td><p>target price</p></td>
<td><p>double</p></td>
</tr>
<tr class="odd">
<td><p>managerApproval</p></td>
<td><p>approved</p></td>
<td><p>Boolean</p></td>
</tr>
</tbody>
</table>

4. When you’ve added the fields, save the data object by clicking on the *Save* button in the top menu.
5. Use the \_breadcrumb\` navigator at the top-left of the screen to navigate back to our `order-management` project.
6. Click on the blue *Add Asset* button in the top-right corner and create a new *Data Object*. Give it the name `SupplierInfo`

![](images/new-data-object-supplier-info.png)

7. Give the `SupplierInfo` object the following fields:

<table>
<colgroup>
<col style="width: 33%" />
<col style="width: 33%" />
<col style="width: 33%" />
</colgroup>
<tbody>
<tr class="odd">
<td><p>Identifier</p></td>
<td><p>Label</p></td>
<td><p>Type</p></td>
</tr>
<tr class="even">
<td><p>offer</p></td>
<td><p>best offer</p></td>
<td><p>double</p></td>
</tr>
<tr class="odd">
<td><p>deliveryDate</p></td>
<td><p>delivery date</p></td>
<td><p>Date</p></td>
</tr>
<tr class="even">
<td><p>user</p></td>
<td><p>user</p></td>
<td><p>String</p></td>
</tr>
</tbody>
</table>

8. We’re done creating our data model.

![](images/data-model-created.png)

We can now start with our process design.

Process Design
--------------

With the domain model defined, we can now sketch out the main flow of the process, the actors, the user task nodes and the required automation decisions.

![](images/order-management-process.png)

1.  Create a new `Business Process` asset. Name it `OrderManagement`.

2.  When the process designer opens, scroll down in the property panel on the right side of the screen, until you see the section *Process Data*.
    
3. Expand the *Process Data* section and add the following 3 *Process Variables* by clicking on the *+* sign.

   <table>
   <colgroup>
   <col style="width: 50%" />
   <col style="width: 50%" />
   </colgroup>
   <tbody>
   <tr class="odd">
   <td><p>Name</p></td>
   <td><p>Data Type</p></td>
   </tr>
   <tr class="even">
   <td><p>orderInfo</p></td>
   <td><p>OrderInfo</p></td>
   </tr>
   <tr class="odd">
   <td><p>supplierInfo</p></td>
   <td><p>SupplierInfo</p></td>
   </tr>
   <tr class="even">
   <td><p>approved</p></td>
   <td><p>Boolean</p></td>
   </tr>
   </tbody>
   </table>
   ![](images/process-variables.png) 

   4. In the palette on the left-side of the editor, select the `Lane` component:

   ![palette-swimlane](images/palette-swimlane.png) 

   5. Create the following 3 swimlanes:  **Supplier** , **Purchase** ,  **Manager**

   ![](images/three-swimlanes.png)

   6. Create the **Start Event** node in the `Purchase` swimlane.
   7. Create the `Prepare Offer` **User Task** node in the `Supplier` swimlane and connect it to the **Start Event** node. Set the following properties on the node via the properties panel on the right side of the screen:
      * Task Name: `PrepareOffer`
      * Subject: `Prepare Offer for #{orderInfo.item}`
      * Actors: `#{supplierInfo.user}`
      * Assignments:

   <table>
   <caption>Data Inputs and Assignments</caption>
   <colgroup>
   <col style="width: 33%" />
   <col style="width: 33%" />
   <col style="width: 33%" />
   </colgroup>
   <tbody>
   <tr class="odd">
   <td><p>Name</p></td>
   <td><p>Data Type</p></td>
   <td><p>Source</p></td>
   </tr>
   <tr class="even">
   <td><p>orderInfo</p></td>
   <td><p>OrderInfo</p></td>
   <td><p>orderInfo</p></td>
   </tr>
   <tr class="odd">
   <td><p>supplierInfo</p></td>
   <td><p>SupplierInfo</p></td>
   <td><p>supplierInfo</p></td>
   </tr>
   </tbody>
   </table>

   <table>
   <caption>Data Outputs and Assignments</caption>
   <colgroup>
   <col style="width: 33%" />
   <col style="width: 33%" />
   <col style="width: 33%" />
   </colgroup>
   <tbody>
   <tr class="odd">
   <td><p>Name</p></td>
   <td><p>Data Type</p></td>
   <td><p>Target</p></td>
   </tr>
   <tr class="even">
   <td><p>supplierInfo</p></td>
   <td><p>Supplier Info</p></td>
   <td><p>supplierInfo</p></td>
   </tr>
   </tbody>
   </table>
   ![](images/prepare-offer-node.png)

   

8. Create the `Auto Approve Order` **Business Rule** node in the `Purchase` swimlane and connect it to the `Prepare Offer` node. Set the following properties:
   * Rule language: `DMN`
   * Assigments:

<table>
<caption>Data Inputs and Assignments</caption>
<colgroup>
<col style="width: 33%" />
<col style="width: 33%" />
<col style="width: 33%" />
</colgroup>
<tbody>
<tr class="odd">
<td><p>Name</p></td>
<td><p>Data Type</p></td>
<td><p>Source</p></td>
</tr>
<tr class="even">
<td><p>Order Information</p></td>
<td><p>OrderInfo</p></td>
<td><p>orderInfo</p></td>
</tr>
<tr class="odd">
<td><p>Supplier Information</p></td>
<td><p>SupplierInfo</p></td>
<td><p>supplierInfo</p></td>
</tr>
</tbody>
</table>

<table>
<caption>Data Outputs and Assignments</caption>
<colgroup>
<col style="width: 33%" />
<col style="width: 33%" />
<col style="width: 33%" />
</colgroup>
<tbody>
<tr class="odd">
<td><p>Name</p></td>
<td><p>Data Type</p></td>
<td><p>Target</p></td>
</tr>
<tr class="even">
<td><p>Approve</p></td>
<td><p>Boolean</p></td>
<td><p>approved</p></td>
</tr>
</tbody>
</table>

![](images/auto-approval-rules-node.png)

-----

*After we’ve created our DMN Decision Model, we will revisit the configuration of this node to reference this DMN model via it’s `name` and `namespace` properties.*

----

9. Create an **X-OR Gateway**/**Exclusive Gateway** in the `Manager `swimlane, below the `Auto Approve Order` node and connect it to that node.

![](images/xor-gateway-auto-approve-order.png)

10. Create the `Approve` **User Task** in the `Manager` swimlane and connect it to the **X-OR** gateway. Set the following properties:
    * Task Name: `Approve`
    * Subject: `Approve Order of #{orderInfo.item}`
    * group: `rest-all`
    * Assignments:

<table>
<caption>Data Inputs and Assignments</caption>
<colgroup>
<col style="width: 33%" />
<col style="width: 33%" />
<col style="width: 33%" />
</colgroup>
<tbody>
<tr class="odd">
<td><p>Name</p></td>
<td><p>Data Type</p></td>
<td><p>Source</p></td>
</tr>
<tr class="even">
<td><p>orderInfo</p></td>
<td><p>OrderInfo</p></td>
<td><p>orderInfo</p></td>
</tr>
<tr class="odd">
<td><p>supplierInfo</p></td>
<td><p>SupplierInfo</p></td>
<td><p>supplierInfo</p></td>
</tr>
</tbody>
</table>

<table>
<caption>Data Outputs and Assignments</caption>
<colgroup>
<col style="width: 33%" />
<col style="width: 33%" />
<col style="width: 33%" />
</colgroup>
<tbody>
<tr class="odd">
<td><p>Name</p></td>
<td><p>Data Type</p></td>
<td><p>Target</p></td>
</tr>
<tr class="even">
<td><p>orderInfo</p></td>
<td><p>OrderInfo</p></td>
<td><p>orderInfo</p></td>
</tr>
</tbody>
</table>

![](images/approve-node.png)



11. Create an **X-OR Gateway**/**Exclusive Gateway** in the `Manager` swimlane, after the `Approve` node and connect it to that node.

![](images/xor-gateway-approve.png)



12. Create another **X-OR Gateway**/**Exclusive Gateway** under the `Manager` swimlane (so outside of the swimlane) and connect it to the two other **X-OR Gateways**/**Exclusive Gateways** as shown in image below:

![](images/xor-gateway-approved.png)



13. Create the `Place Order in ERP` **Script Task** under the `Manager` swimlane (so outside of the swimlanes) and connect it to the **X-OR Gateway** we created earlier. Set the following script in the node’s properties properties:

```
System.out.println("Place Order in ERP");
```

![](images/place-order-in-erp-node.png)



13. Create an **End Event** node under the `Manager` swimlane (so outside of the swimlanes) and connect it to the `Place Order in ERP` node. Name it `Approved`.

![](images/approved-end-event.png)



14. Create an **End Event** node in the `Purchase` swimlane and connect it to the **X-OR Gateway**. Name it `Rejected`.

![](images/rejected-end-event.png)



15. On the **Sequence Flow** from the **X/OR Gateway** before the `Approve` node that is connnected ot the other **X/OR Gateway**, set the following condition, which tells the process engine that this path should be taken when the order is not automatically approved:
    * Process Variable: `approved`
    * Condition: `Is true`

![](images/sequence-flow-approved-condition.png)



16. On the **Gateway** before the `Approve node`, set the **Default Route** property to `Approve`.

![](images/xor-gateway-default-route-approve.png)



17. On the **Sequence Flow** from the **X/OR Gateway** after the `Approve` task, which is connected to the **X/OR Gateway** before the `Place Order in ERP` task, set the following condition:
    * Process Variable: `orderInfo.managerApproval`
    * Condition: `Is true`

![](images/sequence-flow-manager-approval-condition.png)

18. On the **X/OR Gateway** after the `Approval` node , set the **Default Route** to `Rejected`.

![](images/xor-gateway-default-route-rejected.png)

19. Save the process definition.

![](images/completed-process.png)

With the overall layout of the process definition complete, the routing logic implemented, and the I/O assignments defined, we can now implement the business rules of our automated approval decision.

Business Rules and Decisions
----------------------------

Our **Order Management** process contains a **Business Rule Task**, but we have not yet defined the *Decision Model* that will be used in the task. In this paragraph we will implement the automatic approval rules in the form of a DMN model.

1.  In the main project page, the so called **library view**, click on the **Add Asset** button.
    
2. In the next screen, set the drop-down filter to **Decision**. Select the **DMN** asset. Give it the name `order-approval`.

   ![](images/add-asset-dmn-order-approval.png)

   3. In the DMN editor, open the property-panel on the right-side of the screen and set the
      *Namespace* property to: `http://www.redhat.com/dmn/demo/order-management-dmn` . First we need to import our data-model, so we can use it in our DMN decisions. In the DMN editor, click on the *Data Types* tab and click on the *Import Data Object* button at the right-hand side of the screen:

   ![](images/dmn-import-data-object.png) 

   4. Select both the `OrderInfo` and `SupplierInfo` objects and click on the *Import* button:

   ![dmn-import-data-objects-select.png[]](images/dmn-import-data-objects-select.png[].png)

   

   ​	With the 2 datatypes imported, we need to create a third type that will hold the possible values for the `urgency` field of our `Order Information`. Click on the blue *Add* button in the top-right corner. 

   5. In the entry that opens, give the data type the *Name* `Urgency` and the *Type* `string`:

   ![](images/dmn-data-type-urgency.png) 

   6. Click on the *Add Constraints* button, select `Enumeration` as the *constraint type*, and set the values  low` and `high`.

   ![](images/dmn-data-type-urgency-enumeration.png) 

   

   7. Click on the blue checkmark button to save the type.

   ![](images/dmn-datatype-save.png) 

   

   8. Navigate back to the model via the *Model* tab. Add 2 `Input` nodes to the model and name them `Order Information` and `Supplier Information`

   ![](images/dmn-input-nodes.png) 

   

   9. Select the `Order Information` node. Open the properties panel on the right-hand side of the screen, and set the *Data type* to `OrderInfo`.

   ![](images/dmn-orderinfo-datatype.png) 

   10. Do the same for the `Supplier Information` node. Set the *Data type* to `SupplierInfo`. Create a new `Business Knowledge Model` node, name it `Price Tolerance`.

   11. Click on the node, and click on the *Edit* button to start editting the node:

   ![](images/dmn-bkm-price-tolerance-edit.png)

   ![](images/dmn-bkm-price-tolerance-feel-function.png) 

   12. Click in the *Edit parameters*. An editor will open. Click on *Add parameter*. Name the parameter `order information` and set the type to `OrderInfo`.

   ![](images/dmn-bkm-price-tolerance-feel-function-parameter.png) 

   13. Right click in the empty white cell under the parameter definitions and select *Clear*. The text *Select expression* will appear in the cell. Click on the cell and select `Decision Table`.

   ![](images/dmn-bkm-price-tolerance-decision-table.png) 

   14. Add an *input clause* to the decision table. The name of the *input clause* is `order information.urgency`, which references the `urgency`  attribute of the `order information` parameter. Set the type to `Urgency`, which references the `Urgency` enumeration we created earlier.

   ![](images/dmn-bkm-price-tolerance-dtable-input-clause.png) 

   15. Set the *output clause* data type to `number`. Leave the name empty.

   ![](images/dmn-bkm-price-tolerance-dtable-output-clause.png) 

   16. Click on the `Price Tolerance` cell (top cell of the table), and set the data type to `number`.

   ![](images/dmn-bkm-price-tolerance-data-type.png) 

   17. Implement the rest of the decision table as shown below. And save the DMN model.

   ![](images/dmn-bkm-price-tolerance-dtable-complete.png) 

   

   18. Navigate back to the model by clicking on the *Back to order-approval* link at the top-left of the editor.   Create a new *Decision Node* and name it `Approve`. Connect the 2 input nodes and out `Price Tolerance` busines knowledge model node to the new decision node.

   ![](images/dmn-complete.png) 

   

   19. Select the `Approve` decision node and click on the edit button.

   ![](images/dmn-decision-node-approve-edit.png)

   20. Click on \_Select Expression, and set the logic type to `Literal Expression`.
       ![](images/dmn-decision-node-approve-literal-expression-type.png)

       

   21. Enter the following expression: `Supplier Information.offer < Price Tolerance(Order  Information) * Order Information.targetPrice`

   ![](images/dmn-decision-node-aprove-literal-expression.png)

   

   22. Click on the `Approve` cell (top cell of the table), and set the data type to `boolean`.

   ![](images/dmn-decision-node-approve-type.png) 

   23.  Navigate back to the model by clicking on the *Back to order-approval* link at the top-left of the editor. 
   24. Our DMN model is now complete. Make sure to save your model. 
   25. With our DMN model implemented, we can now revisit our **Business Rules Task** in our BPMN2 model. Open the `order-management` process definition and click on the`Auto Approval Order` node. 
   26. Open the node’s properties in the property-panel on the right side of the editor, open the
       **Implementation/Execution** section and set: 
       * Namespace: `http://www.redhat.com/dmn/lab/order-approval-dmn` 
       * Name:`order-approval` 
   27.  In the same properties panel, expand the *Data Assignments* section and open the *Assignments* editor
   28. Implement the following data input and output assignments.

   ![](images/decision-auto-approve-order-data-io.png) 

   29. Our BPMN model is now complete. Make sure to save the model.

Now, we should create implement our forms.

Forms
-----

In this section we are going to create the process start and user-task forms. We could simply generate these forms with the click of a button, which gives us some standard forms based on the process and task data. 

In this lab however, we will be creating these forms using the **Form Modeler** tool. This allows us to design these forms to our specific needs.

Let’s start with the process start form. We want to create the following form:

![OrderManagement Task Form](images/OrderManagement-taskform-form.png)

1.  In the project’s library view, click on **Add Asset**. Filter on **Form**, click on the **Form** tile. Enter the details as shown in the screenshot below:
    
    ![Create Process Start Form](images/create-process-start-form.png)

2.  On this form we want to specify the initial order. We therefore require fields from the `orderInfo` and `supplierInfo` process variable. When we expand the `Model Fields` section, we can see our 2 process variables (`orderInfo` and `supplierInfo`). These are both complex objects. To work with complex objects (as opposed to simple types like integers and booleans), we require a data-form for that specific object. We therefore first need to create a data-form for our `OrderInfo` and `SupplierInfo` objects.
    
3.  Go back to the project’s library view, click again on **Add Asset** and create a new form. Use the following details:

![Create Order Management Order
Form](images/create-order-management-order-form.png)

4. Using the Form Modeler constructs, create the following form:

![OrderManagement Order](images/OrderManagement-Order-form.png) 

5. To create this form, drag both the `item`, `urgency` and `targetPrice` onto the canvas and configure them as follows.

   * List Box:

![Item Name OrderManagement-Order
Field](images/item-name-ordermanagement-order-field.png)

   * Radio Group:

![Urgency OrderManagement-Order
Field](images/urgency-ordermanagement-order-field.png)

   * Decimal Box:

![Target Price OrderManagement-Order
Field](images/target-price-order-management-order-field.png)

6. Save the form and create another new form for our `supplierInfo`. Use the following details.

![](images/create-order-management-supplierinfo-form.png).



7. Using the Form Modeler constructs, create the following form:

![OrderManagement
SupplierInfo](images/OrderManagement-SupplierInfo-form.png) 

8. To create this form, drag the `user` field onto the canvas and configure it as follows.

![User OrderManagement-SupplierInfo
Field](images/user-ordermanagement-supplierinfo-field.png)

9. Save the form and open the `OrderManagement` form (the first form we created).
10. Drag the `orderInfo` process variable onto the canvas. In the pop-up form, set the `OrderManagement-Order` form we just created as the **Nested Form**:

![Order Management Order Subform
Select](images/ordermanagement-order-subform-select.png) 11. Drag the `supplierInfo` process variable ontoo the canvas. In the pop-up form, set the `OrderManagement-SupplierInfo` form we just created as the **Nested Form**:

![Order Management SupplierInfo Subform
Select](images/ordermanagement-supplierinfo-subform-select.png)

---

Next, we will create the form for the `Prepare Offer` **User Task**.

1.  Create a new form. Provide the following details:

    ![Create Prepare Offer Task
    Form](images/create-prepare-offer-task-form.png) 2.  Our aim is to create a form that looks as such:
    
![PrepareOffer Task Form](images/PrepareOffer-taskform.png) 
    
    3.  As with the process start form, this user-task form operates on 2 variables, `orderInfo` and `supplierInfo`. And, as with the process start form, we need to create a data-object form for each of these variables. Technically, data-object forms for a certain data-object can be reused in multiple task-forms. However, creating a data-object form per task-form allows us to design these data-object forms aimed for that specific task.

**PrepareOffer-OrderInfo** ![PrepareOffer
OrderInfo](images/PrepareOffer-OrderInfo-form.png)

**PrepareOffer-SupplierInfo** ![PrepareOffer
SupplierInfo](images/PrepareOffer-SupplierInfo-form.png)



---

Finally, we need to create the task form for the `Approve` task.

1.  Create a new form. Provide the following details.

    ![Create Approve Task Form](images/create-approve-task-form.png) .
    
2. Our aim is create a form that looks like this:
    
    ![Approve Task Form](images/Approve-taskform.png) 
    
    
    
    3. As with the other forms, this user-task form operates on 2 variables, `orderInfo`, `supplierInfo`. And, as with the other forms, we need to create a data-object form for each of these variables.

**Approve-SupplierInfo** ![Approve SupplierInfo
SupplierInfo](images/Approve-SupplierInfo-form.png)

**Approve-OrderInfo** ![Approve OrderInfo
Form](images/Approve-OrderInfo-form.png)



**Don’t forget to save all your forms!!!**

The implementation of our process is complete. It’s now time to deploy and test our application.

Deploying the Process Service
=============================

With our **Order Management** project’s process, decisions and forms completed, we can now package our project in a Deployment Unit (KJAR) and deploy it on the Execution Server. To do this:

1.  Go back to our project’s Library View (for example by clicking on the `Order Management` link in the breadcrumb navigation in the upper-left of the screen).
    
2.  Click on the **Deploy** button in the upper-right corner of the screen. This will package our project in a Deployment Unit (KJAR) and deploy it onto the Execution Server (KIE-Server).
    
3.  Go to the **Execution Servers** perspective by clicking on "Menu → Deploy → Execution Servers". You will see the **Deployment Unit** deployed on the Execution Server.

Execute the process
===================

In this section, you will execute the process deployed on the Process Execution Server via the Business Central workbench.

1.  Navigate to **Menu → Manage → Process Definitions**. If everything is correct, the `order-management` process will be listed. Click on the kebab icon of the `order-management` process and click on **Start**.
    
    ![Start Process](images/start-process.png)
    
2.  In the form that opens, pick the **Huawei P10 Phone** as the item and set the urgency to **low**. Set the target price to **700** and set the supplier name to the name of your own Business Central user (e.g. `pamAmdmin`). Click on **Submit**.
    
    ![](images/start-process-form.png) 3. In the process instance details screen that opens, click on the **Diagram** tab to open the process instance diagram, which shows the current state of the process.
    
![Start Process](images/process-instance-state-diagram-prepare.png)

4. The process is in a wait state at the `Prepare Offer` task. Navigateto **Menu → Track** Task Inbox\*\*. Click on the `Prepare Offer` task to open its task window.
5. Click on the **Start** button to start working on the task. Because the task has been assigned to a single user (via \#{supplierInfo.user}), you don’t have to first *claim* the task.
6. Select a random delivery date. Set the best offer to **900**. Click on **Complete**.![Prepare Offer Complete
   Task](images/prepare-offer-complete-task.png)

7. The process will continue to the `Auto Approve Order` decision node. Because of the target prices set, and the offered price, the decision will evaluale to `false`. Hence, the process will continue to the `Approve` task.

![Start Process](images/process-instance-state-diagram-approve.png)

8. Go back to the **Task Inbox** and open the `Approve` task. Click on **Claim** and on **Start**. In this form we can approve or disapprove the order via the **approved** checkbox, and specify a **rejection reason** if we reject the order. Approve the task by checking the **approved** checkbox and clicking on **Complete**:

![Approve Complete Task](images/approve-complete-task.png)

9. Go back to the process instance view and observe that the process instance is gone.

10. Enable the **Completed** checkbox in the **State** filter on the left-hand-side of the screen. Observe that we can see our process instance in the list.

11. Open the process instance, open it’s **Diagram** tab. Observe that the order has been accepted:

![Process Instance Completed Order
Accepted](images/process-instance-completed-order-accepted.png)

Run a couple more process instances with different values to test, for example, the functionality of the `Automated Approval Rules`.

Correcting problems and errors
==============================

During process instance execution, a lot of things can go wrong. Users might fill in incorrect data, remote services are not available, etc. In an ideal world, the process definition takes a lot of these possible problems into account in its design. E.g. the process definition might contain exception handling logic via boundary catching error events and retry-loops. However, there are situations in which an operator or
administrator would like to manually change the process to another statem for example, restart an already completed **User Task**. In the latest version of Red Hat Process Automation Manager this is now possible via the **Process Instance interface** in Business Central.

1.  Start a new process instance of our **Order Management** process.

2.  Complete the `Prepare Offer` task in such a way that the order is not automatically approved and the process will hit the `Approve` *User Task* wait state.
    
3.  Go to the **Process Instances** view and select the process instance. Navigate to the **Diagram** tab.  Observe that the process is waiting in the `Approve` **User Task**.
    
4.  Click on the `Prepare Offer` node to select it. In the **Node Actions** panel on the left-hand-side of the screen, verify that the `Prepare Offer` node is selected and click on **Trigger**. Observe that the `Prepare Offer` **User Task** has been activated.
    
5. Although we have re-activated the `Prepare Offer` node, we have not yet de-activated the `Approve` task. Click on the active `Approve` task and expand the **Node Instances** section in the **Node Actions** panel. Click on the kebab icon of the active `Approve` instance and click on **Cancel**:

   ![](images/approve-node-cancel.png) 6. Open the **Task Inbox**. Observe that the `Approve` **User Task** is gone and that we have a new `Prepare Offer` task. 

   * Open the `Prepare Offer` task, set the price to a price which will trigger the rules to automatically
     approve the order, and complete the task. . Go to the process instances view and observe that the process instance has been completed. . Enable the **Completed** filter in the **State** filter panel on the left-hand-side of the screen. Open the completed process instance and open its *Diagram* tab.

   ![Process Instance Completed Order
   Accepted](images/corrected-process-instance-completed.png)

Execute the process via APIs
============================

The Execution Server provides a rich RESTful API that allows user to interact with the process engine and deployed processes via a REST. This powerful feature allows users to create modern user interface and
applications in their technology of choice (e.g. Entando DXP, ReactJS/Redux, AngularJS, etc.) and integrate these applications with the process engine to create modern, process driven, enterprise applications.

The Swagger interface provides the description and documentation of the Execution Server’s RESTful API. At the same time, it allows the APIs to be called from the UI. This enables developers and users to quickly test a, in this case, a deployed business process.

1.  Navigate to the [KIE Server Swagger Page](http://localhost:8080/kie-server/docs)
    
2.  Locate the **Process instances** section. The Process Instances API provides a vast array of operations to interact with the process engine.
    
3.  Locate the **POST** operation for the resource `/server/containers/{containerId}/processes/{processId}/instances`. This is the RESTful operation with which we can start a new process instance. Expand the operation:
    
    ![](images/start-process-instance-rest-operation.png)
    
4.  Click on the **Try it out** button.

    -   Set the **containerId** to `order-management` (in this case we
        use the alias of the container).

    -   Set the **processId** to `order-management.OrderManagement`.

    -   Set **Parameter content type** to `application/json`.

5. Set the **Response content type** to `application/json`.

   -   Set the **body** to:

       ````json
       { 
         "orderInfo" : {
       		"com.myspace.order\_management.OrderInfo" : { 
             "item" : "Huawei P10", "urgency" : "low", "price" : 0.0, "targetPrice": "700.0" 
         	}
   	}, 
         "supplierInfo" : {
       		"com.myspace.order\_management.SupplierInfo" : { "user" : "pamAdmin" } 
          } 
       }
       ````
       
       ![Process Instance Rest
       Request](images/send-new-process-instance-rest-request.png)

6.  Click on the *Execute* button.

7.  If requested, provide the username and password of your **Business Central** and **KIE-Server** user (in this example we have been using u: `pamAdmin`, p:`redhatpam1!`).
    
8.  Inspect the response. Note that the operation returns the process instance id of the started process.
    
9. Go back to the Business Central workbench. Go the process instances view and inspect the process instance we have just started.

The RESTful API provides many more operations. Let’s use the API to fetch our **Task List** and complete the `Request Offer` task.

1.  In the Swagger API, navigate to the **Process queries** section.

2.  Find the **GET** operation for the resource `/server/queries/tasks/instances/pot-owners`. Expand the operation and click on the **Try it out** button.
    
3.  Make sure the **\*Response content type** is set to `application/json`. Leave all the other fields set to their default values.
    
4.  Click on the **Execute** button. This will return all the tasks for our user (in the case of this example this is the `pamAdmin` user).
    
![Get Task Instances Potential
    Owner](images/get-task-instances-potential-owner.png)
    
5.  We can see the `Prepare Offer` task that is available in our inbox. Let’s complete this task.
    
6.  Go to the **Task Instances** section in the Swagger interface and locate the **PUT** operation of the `/server/containers/{containerId}/tasks/{taskInstanceId}/states/completed` resource. This is the operation with which we can complete a task.
    
    -   Set the **containerId** to `order-management`.
    
-   Set the **taskInstanceId** to the id of the task instance you want to complete. The task instance id  an be found in the list of task instances we got back from our previous REST operation.
        
-   Set **auto-progress** to `true`. This controls the auto progression of the taks through the various  states of the task lifecycle (i.e. claimed, started, etc.)
        
    -   Set the **Parameter content type** to `application/json`.
    
-   Set the **Response content type** to `application/json`.
    
    -   Set the **body** to:
    
    ````json
        { 
      "supplierInfo": {
            "com.myspace.order\_management.SupplierInfo" : { 
          "user": "pamAdmin", 
              "offer": "900", 
          "deliveryDate":"2020-03-11T12:00:00.000Z" 
            } 
          } 
        }
        ````
        
        
        
    -   Click on the **Execute** button. If you’ve entered everything correctly, the task will be completed and the process will move to the next wait state, the `Prepare Offer` task. . Go back to the Business Central workbench. Go to the process instances view. Select the process instance of the task you’ve just completed. Observe that the `Prepare Offer` task has been completed and that the process is now waiting on the `Approve` **User Task**.



The rest of the tasks can be completed in the same way via the API.

Using the KIE-Server Client
===========================

Red Hat Process Automation Manager provides a KIE-Server Client API that allows the user to interact with  the KIE-Server from a Java client using a higher level API. It abstracts the data marshalling and unmarshalling and the creation and execution of the RESTful commands from the developer, allowing him/her to focus on developing business logic.

In this section we will create a simple Java client for our Order Management process.

1.  Create a new Maven Java JAR project in your favourite IDE (e.g. IntelliJ, Eclipse, Visual Studio Code).
    
2. Add the following dependency to your project:

   ````xml
   <dependency>
   	<groupid>org.kie.server</groupid>
     <artifactId>kie-server-client</artifactId>
     <version>7.30.0.Final</version>
     <scope>compile</scope>
   </dependency>	
   ````

   

3. Create a Java package in your `src/main/java` folder with the name`com.myspace.order_management`.

4. Download the `OrderInfo.java` file from [this](https://raw.githubusercontent.com/DuncanDoyle/order-management-rhpam-lab-client/master/src/main/java/com/myspace/order_management/OrderInfo.java) location and add it to the package you’ve just created.

5. Download the *SupplierInfo.java* file from [this](https://raw.githubusercontent.com/DuncanDoyle/order-management-rhpam-lab-client/master/src/main/java/com/myspace/order_management/SupplierInfo.java) location and add it to the package.

6. Create a new Java class called `Main`. Add a `public static void main(String[] args)` method to your main class.

7. Before we implement our method, we first define a number of constants that we will need when implementing our method (note that the values of your constants can be different depending on your environment, model namespace, etc.):

   ```java
   private static final String KIE_SERVER_URL = "http://localhost:8080/kie-server/services/rest/server"; 
   private static final String CONTAINER_ID = "order-management"; 
   private static final String USERNAME = "pamAdmin"; 
   private static final String PASSWORD = "redhatpam1!"; 
   private static final String PROCESS_ID = "order-management.OrderManagement";
   ```

   

8. KIE-Server client API classes can mostly be retrieved from the `KieServicesFactory` class. We first need to create a `KieServicesConfiguration` instance that will hold our credentials and defines how we want our client to communicate with the server:

   ``` java
   KieServicesConfiguration kieServicesConfig = KieServicesFactory.newRestConfiguration(KIE_SERVER_URL, new EnteredCredentialsProvider(USERNAME, PASSWORD)); 
   ```

   

9. To allow the KIE-Server Client’s marshaller to marshall and unmarshall instances of our domain model, we need to add our domain model classes to the `KieServicesConfiguration`. 

   ``` java
   Set<Class<?>> extraClasses = new HashSet<>();
   extraClasses.add(OrderInfo.class);
   extraClasses.add(SupplierInfo.class);
   kieServicesConfig.addExtraClasses(extraClasses);
   ```

   

10. Next, we create the `KieServicesClient`:

    ``` java
    KieServicesClient kieServicesClient = KieServicesFactory.newKieServicesClient(kieServicesConfig);
    ```

    

11. From this client we retrieve our `ProcessServicesClient`:

    ``` java
    ProcessServicesClient processServicesClient = kieServicesClient.getServicesClient(ProcessServicesClient.class);
    ```

    

12. We now create a `Map` which we will use to pass the process input variables. We create a new `OrderInfo` instance and `SupplierInfo` instance and put them in the `Map`.

    ``` java
    Map<String, Object>inputData = new HashMap<>();
    OrderInfo orderInfo = new OrderInfo();
    orderInfo.setItem("Huawei P10");
    orderInfo.setUrgency("low");
    inputData.put("orderInfo", orderInfo);
    
    SupplierInfo supplierInfo = new SupplierInfo();
    supplierInfo.setUser("pamAdmin"); inputData.put("supplierInfo", supplierInfo); 
    ```



13.  We can now start a new process instance via the `ProcessServicesClient`.

````java
Long processInstanceId = processServicesClient.startProcess(CONTAINER_ID, PROCESS_ID, inputData);
````

14. Finally, we can print the process instance id to `System.out`.

``` java
System.out.println("New *Order Management* process instance started with instance-id: " + processInstanceId); 
```

15. Compile your project and run it. Observe the output in the console, which should say: **New *Order Management* process instance started with instance-id**

----

The complete project can be found here: <https://github.com/DuncanDoyle/order-management-rhpam-lab-client>

The KIE-Server Client provides more services to interact with the Execution Server:

-   `UserTaskServicesClient`: provides functionality to interact with the **UserTask** services, for example to claim, start and complete a **User Task**.
    
-   `CaseServicesClient`: provides functionality to interact with the **Case Management** features of the Execution Server.
    
-   `ProcessAdminServicesClient`: provides the administration API for processes.
    
-   etc.

We leave as an exercise to the reader to try to complete a **User Task**, of the process instance we’ve just created, using the `UserTaskServicesClient`.
