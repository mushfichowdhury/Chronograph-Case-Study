# Chronograph-Case-Study
## PART 1

**Assumptions**

We’ve created a Postgres database which includes reports, documents and pages tables. reports has a one-to-many relationship with documents, and documents has a one-to-many relationship with pages. The database schema is outlined in Appendix A. Additionally, you may assume:

● All non-primary key database columns default to null.

● All primary and foreign keys are integers.

**Prompt**

Please write a query to answer each of the following questions. Both accuracy and query performance are critical. Please document any assumptions you make in addressing these questions.
<hr>

**1. Write a SQL query to find the ids of all documents which do not have any pages.**
```
SELECT d.id from documents d NOT IN (SELECT DISTINCT document_id FROM pages)
```

Return id’s from document table thats not in the results of the ⇒ return distinctly of document_id values from the pages table.

First SELECT returns all the documents ⇒ 
Second SELECT returns all documents with pages ⇒ 
The NOT IN condition removes the values of the second SELECT from the first SELECT ⇒ 
So from the full list of all documents, I removed all the documents with pages ⇒ 
Which leaves me with all the documents without any pages.

**2. Write a SQL query which returns a list of report titles and the total number of pages in the report. reports which do not have pages may be ignored.**
```
SELECT r.title, COUNT(p.id) FROM reports r JOIN documents d ON d.report_id = r.id JOIN pages p ON p.document_id = d.id GROUP BY r.title HAVING COUNT(p.id) > 0;
```
Return report titles and page counts ⇒ 
Joining reports table and documents table, matching the report id and the documents report_id, returning a reports table which has id, title, corresponding name, and corresponding filetype ⇒ 
Join previous table with pages table, matching the pages document_id to the document id, returning a table with report_id, document_id, page_id, with the corresponding title, name, filetype, body, and footnote ⇒ 
What we need is the report title and page_ids (which we returned in step 1) ⇒ 
GROUP BY r.title will return the page counts for each report title in the table ⇒ 
HAVING COUNT(p.id)>0 ignores reports that do not have pages. 

**3. Assume a new feature needs to be developed to allow commenting on reports, documents, and pages. How would you implement support for this in the schema, and what considerations would you have in determining your approach?**

I would add a new table for comments, with primary key id and foreign keys report_id, document_id, and page_id. This would allow a user to leave multiple comments on any given report/document/page. The comment table would have integers for the four id columns, as well as a varchar column of unspecified length for the comment, to ensure there is ample room for feedback. By specifying varchar, user can type whatever they want and it will trim down any unnecessary blank space. Unlike char which doesn’t trim, you can set varchar counts to 1000, but if you use only 600, it will trim the 400 of blank space.

By creating a separate comment table, we avoid creating a potential many-to-many relationship among the three report/document/page tables, and this would be the more normalized form. In a relational database you want to avoid a many to many relationship because it will create data redundancies when you query data. The best practice is to create a bridge table, so instead of a many to many, you create a third table which creates two one to many’s which can be queried easier. Comments is one to many for documents, pages, and reports. Each report has many comments, same with documents and pages.


## PART 2

**Assumptions** 

document, page, and report entities are represented as objects in our front-end data store. The store is a single object whose structure may be ascertained from the example in Appendix B. You may assume that entity objects are always keyed by id.

**Prompt**

Given a store like the example in Appendix B, please answer the following questions using vanilla ES6+ Javascript. You may define and reuse auxiliary functions to aid your responses.
<hr>
**1. Return an object which maps report_ids to the total number of pages in the report.**

```
function getPages(obj) {
  let documents = obj['document']
  let pages = obj['page']
  let reports = obj['report']
  let reportKeys = []
  let documentKeys = {}
  let results = {}
  for (var key in reports) {
    reportKeys.push(reports[key]['id'])
    results[reports[key]['id']] = 0
  }
  for (var key in documents) {
    if (reportKeys.includes(documents[key]['report_id'])) {
      documentKeys[documents[key]['id']] = documents[key]['report_id']
    }
  }
  for (var key in pages) {
    if (documentKeys.hasOwnProperty(pages[key]['document_id'])) {
      let docKey  = documentKeys[pages[key]['document_id']]
      results[docKey] += 1
    }
  }
}

getPages(store)
```

**2. Please write a search function which accepts a string and returns a list of reports. Any string field in our schema may contain relevant text matches, excluding documents.filetype.**
```
function searchReport(store, str) {
  str = str.toLowerCase()
  const results = [];
  for (const key in store) {
    for (const secondKey in store[key]) {
      if (typeof store[key][secondKey] == "object") {
        for (const thirdKey in store[key][secondKey]) {
          let text = store[key][secondKey][thirdKey];
          if (typeof text == "string" && text.toLowerCase().includes(str)) {
            results.push(store[key][secondKey]);
          }
        }
      }
    }
  }
  return results;
}
```

**3. We’ve replaced your solution from part (2) with an asynchronous search function which loads its data from an API.**

a. Ignoring the body of the function, how would its signature change? Feel free to propose multiple options, and demonstrate how the asynchronous function could be used to fetch search results.

I don't believe the function's signature would change too drastically. Input arguments would still need a string and the return type would remain the same. Since we are now fetching the data from an API, we would need to have an async/await function to save the data to a variable, similar to how we have the store object. Once the data is successfully returned, we can search similar to the previous iteration of the function, as long as the structure of the data is similar. 

b. If the asynchronous function can produce errors, how would those be handled?
	
You can either wrap the function in a try/catch:
```
async function search() {
  try {
    // code that might produce an error (like a rejected promise)
  } catch (err) {
    console.log(err)
  }
}
```
Or you can use conditional rendering to log errors:
```
async function search() {
  if(this.state.data != null) {
    // code
  } else if (typeof this.state.data["document"] == "object") {
    // code
  } else {
    // handle error
  }
}
```


