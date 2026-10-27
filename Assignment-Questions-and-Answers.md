# Assignment Questions and Answers

## 1. What is the difference between Lambda's execution role and Scheduler's execution role?

The Lambda execution role gives the Lambda function permission to access other AWS services.

In this assignment, Lambda #1 uses its execution role to write data to DynamoDB using `PutItem`. Lambda #2 uses its execution role to read data from DynamoDB using `Scan` and write backup files to S3 using `PutObject`.

The Scheduler execution role has a different purpose. It gives Amazon EventBridge Scheduler permission to invoke Lambda #2 automatically every 2 minutes.

So, the Lambda execution role controls what the Lambda function can do, while the Scheduler execution role allows the Scheduler to start the Lambda function.

## 2. Why does running Lambda #1 twice not produce 20 DynamoDB users?

Each user has the same unique `id` every time the JSONPlaceholder API is called.

The DynamoDB table uses `id` as its partition key. When Lambda #1 runs again, `PutItem` writes items using the same IDs from 1 to 10.

Because those IDs already exist, DynamoDB replaces the existing items instead of creating another 10 separate users.

Therefore, the table still contains 10 users after running Lambda #1 twice.

## 3. Why do Lambda #2 backups have unique timestamps and request IDs?

Each backup file needs a unique name so that a new backup does not overwrite an older backup.

The timestamp shows when the backup was created, and the request ID helps make the filename unique even if multiple executions happen close together.

Because of this, every Lambda #2 execution creates a separate JSON file in S3.

## 4. What is the difference between `print()` logs and `Invocations`/`Errors` metrics?

`print()` statements create log messages that are stored in CloudWatch Logs.

These logs help us understand what the Lambda function was doing, such as how many users were saved or where a backup file was created.

`Invocations` and `Errors` are CloudWatch metrics. They show numerical information about the Lambda function, such as how many times it was executed and how many executions failed.

So, logs provide detailed execution information, while metrics provide summarized numerical information about the function.

## 5. What would happen if you deleted the DynamoDB table but kept its JSON files in S3? Could the original indexes/configuration be automatically restored from the JSON alone?

The JSON files contain the exported user records, but they do not contain the complete DynamoDB table configuration.

For example, the JSON backup does not fully preserve the table settings, indexes, IAM permissions, alarms, or other AWS configurations.

Therefore, the data could be used to help recreate the records, but the original DynamoDB table and its complete configuration could not be automatically restored from the JSON files alone.

## 6. Why is an export every 2 minutes usually a poor production backup strategy for large DynamoDB tables?

In this assignment, Lambda #2 scans the entire DynamoDB table every 2 minutes and creates a new JSON backup in S3.

This works well for a small lab table with only a few records, but it is not a good approach for a large production table. If the table contains thousands or millions of items, scanning the full table every 2 minutes would repeatedly read the same data again and again.

This can cause several problems:

- High DynamoDB read usage
- Increased AWS cost
- Longer Lambda execution time
- More S3 storage usage because every run creates another full backup
- Possible Lambda timeout or memory issues for very large tables

A production system should use a more efficient backup method, such as DynamoDB Point-in-Time Recovery (PITR) or DynamoDB Export to S3, instead of performing a full table scan every 2 minutes.

Therefore, the 2-minute scan approach is suitable for this learning assignment, but it would not be efficient or scalable for a real production environment.