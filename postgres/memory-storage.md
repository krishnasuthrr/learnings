## ⭐ How does Postgres actually manage memory and storage?
---

> Memory = RAM
>
> Storage = Disk (HDD, SSD etc)

### **Memory \(RAM\)**
    - Volatile: temporary. data vanishes when computer shuts down
    - fast data access
    - each cluster allocates a dedicated, isolated slice of RAM
    - cache frequently accessed data

### **Disk**
    - non-volatile: permanent. data on disk survives shutdowns
    - larger and cheaper than RAM
    - slower than RAM for R/W
    - the actual data, tables is stored on disk
    - each cluster captures a separate, isolated directory/portion on disk


Postgres only supports **vertical scaling**: increasing physical RAM and Disk Storage


more deepdive later... PGDATA