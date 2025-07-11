This README summarizes  units from the [Multirecord Elements and Transforms in Flows](https://trailhead.salesforce.com/content/learn/modules/multirecord-elements-and-transforms-in-flows?trail_id=build-flows-with-flow-builder) Trailhead module.

# [1. Update and Retrieve Multiple Records](https://trailhead.salesforce.com/content/learn/modules/multirecord-elements-and-transforms-in-flows/update-and-retrieve-multiple-records?trail_id=build-flows-with-flow-builder)

### Installing a package on a Trailhead Playground
1. Open your playground,
2. Click **Install a package** tab,
3. Insert Package ID, click **Install**.

### How to create a record-triggered flow
1. Click **Setup**,
2. Select object from **Object Manager** tab,
3. Click **Flow Triggers** > **New Flow**.

### Otherwise (how to find Flow Builder)
1. **Navigate to Setup**: Click on the gear icon in the top right corner and select "Setup",
2. **Search for Flow**: In the Quick Find box, type "Flows",
3. **Open Flow Builder**: Click on "Flows" under the Process Automation section. From there, you can create a new flow by clicking the "New Flow" button.
(Remember you may need to create some resources like variables etc. You can do this clicking **New Resource** on a left).

### Updating Multiple Related Records
In a record-triggered flow (e.g., when an Opportunity closes lost), use a single **Update Records** element—configured via *Update Related Records*—to automatically update all related records, such as open Cases linked to the same Account.
Key points:
* Apply filters (e.g., only update Cases where Status ≠ Closed),
* Avoid per-record loops—this single-action approach is efficient and avoids hitting governor limits.

### Fetching Multiple Records with Collection Variables
A Get Records element can retrieve multiple records (e.g., all onboarding steps for a Project) and store them in a **record collection variable**.
* Collection variables hold multiple records of the same object type,
* Choosing “All records” + “Automatically store all fields” in Get Records results in a full collection.

# [2. Transform Multiple Records](https://trailhead.salesforce.com/content/learn/modules/multirecord-elements-and-transforms-in-flows/transform-multiple-records?trail_id=build-flows-with-flow-builder)

### Transforms vs. Loops
Flow Builder provides two elements you can use to change every record in a collection together:
* **A Transform element** copies the values of one or more components or variables (single or collection) to an autogenerated variable, while also changing the copy’s finalized values,
* **A Loop element** iterates through multiple values in a collection, one at a time, allowing the flow to run each of those values through a series of elements.
**Transforms** are typically **10× faster** and more scalable, making them the preferred choice for bulk operations. Use loops only when transforms can’t handle the logic required.

### Capturing Selections with a Data Table
Build a screen flow that allows users to choose multiple records using a Data Table component configured with:
* **Multiple row** selection,
* A picklist to select a new value (e.g., new Status).

### Using Transform to Prepare Records
Add a Transform element after the screen:
* Set **Source Data** to the data table’s selected rows,
* Create a new **Target Collection** of the same object type,
* Map fields like `Id`, and use formulas to assign dynamic values (e.g., `{!New_Status}` for status updates).

### Bulk Updating with Update Records
Following the Transform, use a single **Update Records** element:
* Configure it to “Use the IDs and all field values from a record collection",
* Select the collection generated by the Transform to update all records in one go.

### Best Practices & Notes
* Transforms should be your first choice for bulk updates—they’re faster, cleaner, and avoid hitting SOQL limits,
* Reserve loops for scenarios where you need to perform unique logic on each record,
* Visualize data handling: Data Table > Transform (map IDs + set fields via formulas) > Update Records.

# [3. Transform Data From One Form to Another](https://trailhead.salesforce.com/content/learn/modules/multirecord-elements-and-transforms-in-flows/transform-data-from-one-form-to-another?trail_id=build-flows-with-flow-builder)

### Transform Record IDs into Text Collection
* Start with a screen flow where users select records (e.g. Onboarding Project Steps) via a Data Table,
* Use a **Transform** element to map selectedRows’ ~`Id` field into a new text collection,
* This new collection stores record IDs as text strings, enabling compatibility with the In operator.

### Bulk Update Using the In Operator
Add an Update Records element configured to identify records using the In operator:
* **Object**: Onboarding Project Step,
* **Filter**: Record ID In **{!TransformStepIDs}**,
* **Set Fields**: Use the New Status value from the previous screen via the Data Table component.
This enables a single bulk update across all selected records in the text collection.

### Why Use a Text Collection?
The **In** operator in Update Records only supports collections of simple values (like text or numbers), not record collections. Transforming record IDs into a text collection allows efficient filtering and bulk updates.

### How to retrieve source in Manifest from Org (based on Flows)
1. Include `Flow` metadata in `manifest/package.xml`. Adjust inside a `package.xml` following data:

`<types>`

    `<members>*</members>`
    
    `<name>Flow</name>`
    
`</types>`

3. You may not have permission to retrieve all flows, but only you edited or created. Then instead of * between `<members>` and `</members>` you need to write API's name of Flow and repeat. For example:

`<members>Win_Multiple_Opps_Formulas</members>`

`<members>Choose_Steps_to_Update</members>`

4. Right-click the `package.xml` file itself in the **Explorer sidebar** and choose 

`SFDX: Retrieve Source in Manifest from Org`

(you can also do it manually in terminal:

`sf project retrieve start --manifest manifest/package.xml --target-org NameOfMyOrg`.

# [4. Organize Data in a Collection Variable](https://trailhead.salesforce.com/content/learn/modules/multirecord-elements-and-transforms-in-flows/organize-data-in-a-collection-variable?trail_id=build-flows-with-flow-builder)

### Filtering a Collection
Use the **Collection Filter** element to create a new collection containing only items that match specified criteria. The original collection remains unchanged, allowing you to use both filtered and full sets separately.

**Example**: From a list of emails, filter out those from a competitor’s domain.

### Sorting & Truncating a Collection
**Collection Sort** lets you reorder a collection based on a chosen field (e.g., sort Opportunities by Amount descending). Optionally, you can limit the number of items (e.g., keep only the top 10). Unlike filtering, this modifies the same collection, not a copy.

### When to use Filter/Sort vs. Get Records
* Use **Get Records** filters when criteria are known upfront—this is more efficient,
* Use **Collection Filter/Sort** when criteria depend on runtime logic (e.g., decisions or screen inputs).

### Real-World Example
1. **Get Campaign & Opportunities** records (exclude Closed Lost),
2. Use a **Decision** to route by campaign type (e.g., Generators),
3. In one branch, apply **Collection Filter** to select opportunities where `Main_Product_Category = GC`,
4. Then use **Collection Sort** to sort by `Amount` and **truncate** to top 10,
5. Apply **Transform** to convert these filtered & sorted opps into a new Campaign Member collection,
6. Use **Create Records** to insert new campaign members.

Best practice: avoid loops; use **Transform** for performance, then **Create Records** in bulk.
