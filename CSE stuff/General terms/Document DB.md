### Document Databases

- **Structure**: Document DBs, like MongoDB and CouchDB, store data in JSON-like documents. Each document is a self-contained data structure with fields and values, allowing nested structures.
- **Use Case**: Best for applications requiring flexible schemas or complex, hierarchical data (e.g., content management systems, e-commerce product catalogs).
- **Features**: They support querying by fields within the document, indexing, and complex data relationships.
- **Tradeoffs**:
    - **Pros**: Flexible schema, hierarchical data storage, efficient for read-heavy applications with complex querying.
    - **Cons**: Larger storage footprint due to repeated document structures, slower performance for high write-throughput scenarios compared to key-value stores.