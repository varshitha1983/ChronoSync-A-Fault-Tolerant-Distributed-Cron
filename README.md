# ChronoSync-A-Fault-Tolerant-Distributed-Cron

A fault-tolerant and highly available distributed task scheduler designed to manage and execute scheduled jobs reliably across a cluster of machines. This project provides a robust alternative to traditional single-machine cron, ensuring that critical tasks are never missed, even in the event of system failures.

-----

## Key Features

  * **Distributed & Scalable:** Built with **Go** to leverage its native concurrency model, allowing the system to scale horizontally and distribute tasks efficiently across multiple nodes.
  * **Fault Tolerance:** Uses **PostgreSQL** for persistent state management and **row-level locking** (`FOR UPDATE SKIP LOCKED`) to prevent job duplication and guarantee an "at-most-once" execution model.
  * **High Availability:** The system is designed with no single point of failure. If a coordinator or worker node fails, tasks are automatically picked up and processed by other nodes in the cluster.
  * **Robust Communication:** Employs **gRPC** for high-performance and reliable inter-service communication between coordinator and worker nodes.
  * **Efficient Task Execution:** Implements a thread-safe, goroutine-based worker pool to handle a high volume of concurrent tasks without performance degradation.
  * **Containerized:** Packaged with **Docker** for easy setup, consistent development, and simplified deployment.

-----

## Architecture

ChronoSync operates using a classic **Coordinator-Worker** model. The core components work together to ensure tasks are scheduled, managed, and executed with high reliability.

*(Note: Replace the placeholder URL with a link to your actual architecture diagram image)*

### 1\. The Coordinator

The coordinator service is the brain of the system. It runs on multiple nodes and is responsible for:

  * Periodically polling the database for tasks that are due for execution.
  * Using `FOR UPDATE SKIP LOCKED` to safely claim ownership of a batch of tasks, preventing other coordinators from processing the same jobs.
  * Delegating the claimed tasks to available worker nodes via gRPC.

### 2\. The Worker

The worker service is responsible for task execution.

  * It maintains a pool of threads (goroutines) that listen for tasks from a coordinator.
  * Upon receiving a task, the worker executes the command and updates the task's status (start time, completion time, etc.) in the database.

### 3\. PostgreSQL Database

The database serves as the single source of truth for the entire system's state. It stores all scheduled tasks and their execution history. This persistence is crucial for recovering from failures without losing any scheduled jobs.

### 4\. Scheduler

User Task Submission: When a user submits a task, it's a command or script to be run at a specified time. This information is stored in a database table called Tasks.

-----

## Database Schema

The `tasks` table is the heart of the system's state management. We use a detailed, timestamp-based approach instead of a simple status field to ensure consistency and a clear record of task history.

```sql
CREATE TABLE tasks (
    id UUID PRIMARY KEY,
    command TEXT NOT NULL,
    scheduled_at TIMESTAMP WITH TIME ZONE NOT NULL,
    picked_at TIMESTAMP WITH TIME ZONE,
    started_at TIMESTAMP WITH TIME ZONE,
    completed_at TIMESTAMP WITH TIME ZONE,
    failed_at TIMESTAMP WITH TIME ZONE
);
```

-----

##  Getting Started

To get a local instance of Distributed ChronoSync up and running, follow these steps:

1.  **Clone the repository:**

    ```bash
    git clone https://github.com/your-username/distributed-chronosync.git
    cd distributed-chronosync
    ```

2.  **Start the services with Docker Compose:**

    ```bash
    docker-compose up --build
    ```

    This command will build the Docker images for the coordinator and worker services, start a PostgreSQL container, and wire everything together.

3.  **Run a sample task:**
    ```bash
    curl -X POST localhost:8081/schedule -d '{"command":"<your-command>","scheduled_at":"2023-12-25T22:34:00+05:30"}'
    ```
-----

## Contributing

Contributions are welcome\! Please feel free to open a pull request or submit an issue on the repository.
