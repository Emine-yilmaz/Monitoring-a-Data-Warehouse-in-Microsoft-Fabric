# Monitoring a Data Warehouse in Microsoft Fabric

This guide demonstrates how to monitor activity and queries within a Microsoft Fabric data warehouse using Dynamic Management Views (DMVs) and Query Insights.

**Note:** This lab requires a Microsoft Fabric trial or a Fabric-enabled license (Premium or Fabric).

## Prerequisites

* A Microsoft Fabric account with a trial or paid license.
* A web browser to access the Microsoft Fabric portal.

## Steps

### 1. Create a Workspace

1.  Navigate to the [Microsoft Fabric home page](https://app.fabric.microsoft.com/home?experience=fabric) and sign in with your Fabric credentials.
2.  In the left-hand menu, click on **Workspaces** (🗇 icon).
3.  Click **New workspace**.
4.  Provide a name for your workspace.
5.  Ensure that the licensing mode includes Fabric capacity (Trial, Premium, or Fabric).
6.  Click **Create**.

### 2. Create a Sample Data Warehouse

1.  Within your new workspace, click on **Create** in the left-hand menu.
2.  Under the **Data Warehouse** section, select **Sample warehouse**.
3.  Name the new data warehouse `sample-dw`.
4.  Click **Create**.
5.  Wait for a short period while the sample data warehouse is created and populated with data for a taxi ride analysis scenario.

### 3. Explore Dynamic Management Views

1.  On the `sample-dw` data warehouse page, in the **New SQL query** drop-down list, select **New SQL query**.

2.  In the new blank query pane, enter and run the following query to view details of all connections to the data warehouse:

    ```sql
    SELECT * FROM sys.dm_exec_connections;
    ```

    Click the **▷ Run** button to execute the script and view the results.

3.  Modify the SQL code to query the `sys.dm_exec_sessions` DMV and run it to see details of all authenticated sessions:

    ```sql
    SELECT * FROM sys.dm_exec_sessions;
    ```

4.  Modify the SQL code to query the `sys.dm_exec_requests` DMV and run it to view details of all requests being executed:

    ```sql
    SELECT * FROM sys.dm_exec_requests;
    ```

5.  Modify the SQL code to join the DMVs and return information about currently running requests in the same database:

    ```sql
    SELECT connections.connection_id,
           sessions.session_id, sessions.login_name, sessions.login_time,
           requests.command, requests.start_time, requests.total_elapsed_time
    FROM sys.dm_exec_connections AS connections
    INNER JOIN sys.dm_exec_sessions AS sessions
        ON connections.session_id=sessions.session_id
    INNER JOIN sys.dm_exec_requests AS requests
        ON requests.session_id = sessions.session_id
    WHERE requests.status = 'running'
        AND requests.database_id = DB_ID()
    ORDER BY requests.total_elapsed_time DESC;
    ```

    Run the modified query and observe the details of running queries, including the one you just executed.

6.  In the **New SQL query** drop-down list, select **New SQL query** to open a second query tab. In the new tab, run the following infinite loop query:

    ```sql
    WHILE 1 = 1
        SELECT * FROM Trip;
    ```

    Leave this query running.

7.  Return to the first query tab containing the DMV query and re-run it. You should now see the long-running query from the second tab in the results. Note its elapsed time.

8.  Wait a few seconds and re-run the DMV query again. Observe that the elapsed time for the query in the second tab has increased.

9.  Go back to the second query tab where the `WHILE` loop is running and click **Cancel** to stop it.

10. Return to the first query tab and re-run the DMV query to confirm that the second query is no longer running.

11. Close all query tabs.

**Further Information:** For more details on using DMVs for monitoring, refer to the [Monitor connections, sessions, and requests using DMVs](https://learn.microsoft.com/en-us/fabric/data-warehouse/monitor-dmv) documentation in Microsoft Fabric.

### 4. Explore Query Insights

1.  On the `sample-dw` data warehouse page, in the **New SQL query** drop-down list, select **New SQL query**.

2.  In the new blank query pane, enter and run the following query to view details of previously executed queries:

    ```sql
    SELECT * FROM queryinsights.exec_requests_history;
    ```

    Click the **▷ Run** button to execute the script and view the results.

3.  Modify the SQL code to query the `frequently_run_queries` view and run it to see details of frequently executed queries:

    ```sql
    SELECT * FROM queryinsights.frequently_run_queries;
    ```

4.  Modify the SQL code to query the `long_running_queries` view and run it to view details of queries and their durations:

    ```sql
    SELECT * FROM queryinsights.long_running_queries;
    ```

**Further Information:** For more details on using query insights, refer to the [Query insights in Fabric data warehousing](https://learn.microsoft.com/en-us/fabric/data-warehouse/query-insights) documentation in Microsoft Fabric.

### 5. Clean Up Resources

1.  In the left-hand menu, click on your workspace name.
2.  Select **Workspace settings**.
3.  In the **General** section, scroll down and select **Remove this workspace**.
4.  Click **Delete** to confirm the deletion of the workspace.

You have now successfully monitored activity in your Microsoft Fabric data warehouse using both Dynamic Management Views and Query Insights!
