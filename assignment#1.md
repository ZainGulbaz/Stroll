# Dynamic Question Assignment System

## **Database Architecture**

### Table: `Questions`
| Column    | Type        | Description                     |
|-----------|-------------|---------------------------------|
| `id`      | INT (PK, AI) | Primary Key, Auto Incremented   |
| `question`| TEXT        | The question text               |

### Table: `RegionsOffset`
| Column    | Type        | Description                                  |
|-----------|-------------|----------------------------------------------|
| `id`      | INT (PK, AI) | Primary Key, Auto Incremented                |
| `region`  | VARCHAR     | Region name (e.g., USA, Singapore, Europe)    |
| `offset`  | INT         | Offset value for question assignment          |

## **Approach**

The question assignment is based on a region-specific **offset**. Each region has a unique offset that determines which question it will receive during a specific rotation cycle.

- **USA**: `offset = 0`
- **Singapore**: `offset = 1`
- **Europe**: `offset = 2`

Given `N` questions in the `Questions` table, the assignment works as follows:

- In **Cycle 1**:
  - USA gets `Question 1` (`id = 1`)
  - Singapore gets `Question 2` (`id = 2`)
  - Europe gets `Question 3` (`id = 3`)

- In **Cycle 2**:
  - USA gets `Question 2`
  - Singapore gets `Question 3`
  - Europe gets `Question 4`

- In **Cycle N-2**:
  - USA gets `Question N-2`
  - Singapore gets `Question N-1`
  - Europe gets `Question N`

- In **Cycle N-1**:
  - USA gets `Question N-1`
  - Singapore gets `Question N`
  - Europe gets `Question 1`

- In **Cycle N**:
  - USA gets `Question N`
  - Singapore gets `Question 1`
  - Europe gets `Question 2`

This rotation continues for subsequent cycles, ensuring each region receives a new question every cycle.

## **Cron Job**

We will use a **cron job** to manage the rotation. The cron job runs at a configurable interval (e.g., every 7 days) and rotates the assigned questions for each region.

- **Cron Job Execution**: 
  - Every 7th day at a specific time (e.g., Monday 7 pm SGT), the cron job will execute and assign a new question to each region based on the next question in the sequence.

## **User-Region Association**

Each user is associated with a specific region. The system will query the database to retrieve the correct question for each user based on their region and the current cycle.

## **Scalability Considerations**

1. **Indexing**: 
   - Index the columns that are frequently used in `WHERE` clauses or in `JOIN` operations across multiple tables to improve query performance.
   
2. **Sharding by Region**: 
   - To support scalability, user data can be **sharded** based on regions, which will distribute the data across different nodes and improve load balancing and performance for large-scale systems.
