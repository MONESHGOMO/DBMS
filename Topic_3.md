# Storage and File Structures - Theory Notes

## 1. Overview of Physical Storage Media  
Physical storage media refer to hardware devices used to store data permanently or temporarily. Examples include magnetic disks, solid-state drives (SSD), optical discs, and tertiary storage like tape drives.

## 2. Magnetic Disks  
Magnetic disks are common storage devices that store data magnetically on rotating platters. They offer fast random access and are widely used as primary storage in computers.

## 3. RAID (Redundant Array of Independent Disks)  
RAID is a data storage technique that combines multiple physical disks into a single logical unit to improve performance, fault tolerance, or both.

**Purpose:**  
- Increase data reliability through redundancy  
- Improve read/write performance

## 4. Tertiary Storage  
Tertiary storage refers to offline storage devices like magnetic tapes used mainly for backup or archival purposes due to slower access speeds.

## 5. Storage Access  
Storage access refers to how data is retrieved from storage media. It can be:  
- **Sequential access:** Data accessed in order  
- **Direct/Random access:** Data accessed at any location

## 6. File Organization  
File organization defines how records are stored in a file on storage media. Common types include:  
- **Heap file (Unordered):** Records stored randomly  
- **Sorted file:** Records stored in sorted order  
- **Indexed file:** Records stored with indexes for faster access

## 7. Indexing and Hashing  
- **Indexing:** Data structure that improves query speed by maintaining pointers to records.  
- **Hashing:** Technique to compute the address of data using a hash function for direct access.

## 8. Static Hashing  
Uses a fixed number of buckets (storage locations) determined at creation time. Good for stable datasets but poor for dynamic or growing data.

## 9. Dynamic Hashing  
Allows the number of buckets to grow or shrink dynamically with the dataset size, improving efficiency for varying data volumes.
