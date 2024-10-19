# Relate
A CLI tool that lets you Create and Deploy Schema for Relational Databases with ease.

## Usage

You don't need to specify the path to your schema file if you run the commands from the directory where your `relate.edn` file is located.

### `relate apply`
Creates all tables, indexes, and relationships from the `relate.edn` schema.

```bash
relate apply
```

### `relate drop`
Drops all tables and related objects defined in `relate.edn`.

```bash
relate drop
```

### `relate migrate`
Applies any schema changes by comparing the database with `relate.edn`.

```bash
relate migrate
```

### `relate status`
Shows differences between the current database and `relate.edn`.

```bash
relate status
```

### `relate help`
Displays available commands and usage.

```bash
relate help
```

### `relate --file [path]`
Specifies a custom path for the `relate.edn` schema file, allowing you to apply, drop, migrate, or check status using a schema file located at a custom location.

```bash
relate apply --file /path/to/custom_schema.edn
relate drop --file /path/to/custom_schema.edn
relate migrate --file /path/to/custom_schema.edn
relate status --file /path/to/custom_schema.edn
``` 


## Schema Structure

### Connection Configuration

The schema includes basic connection information for your database:

- `:host` - The hostname of the database server (e.g., `localhost`).
- `:dbtype` - The type of database (e.g., `:mysql`).
- `:dbname` - The name of the database you want to connect to (e.g., `:railway_routing_system`).
- `:user` - The user to connect to the database (e.g., `:root`).

```edn
{:host "localhost"                          ; The database host, e.g., localhost or an IP address
 :dbtype :mysql                             ; Database type, e.g., :mysql
 :dbname :railway_routing_system            ; The name of the database
 :user :root}                               ; The database user (ensure credentials are secure)
```

### Tables

The main component of the schema is the list of tables, each represented as a map with the following keys:

- `:name` - The name of the table.
- `:columns` - A list of columns defined by `[name data-type [optional-constraints]]`.

#### Each column has:

- `name` - The name of the column (e.g., `:id`).
- `data-type` - The data type for the column (e.g., `:varchar`, `:char`, `:decimal`).

- `:primary-key` - The list of columns that make up the primary key of the table.
- `:foreign-keys` (optional) - A list of foreign key constraints to define relationships between tables.

#### Each foreign key includes:

- `:column` - The name of the column with the foreign key.
- `:references` - A vector specifying the referenced table and column.

### Example Table Definitions

This table, named `train`, has columns such as `:id`, `:number`, and `:capacity`. The primary key is defined as the `:id` column.

```edn
{:tables [{:name :train
           :columns [[:id :char 36]                    ; Unique train ID
                     [:number :varchar 50]             ; Train number
                     [:type :varchar 50]               ; Type of train
                     [:capacity :int]                  ; Capacity of passengers
                     [:status :varchar 50]]            ; e.g., 'active', 'maintenance'
           :primary-key [:id]}]}                       ; Primary key is the :id column
```

### Foreign Keys

Foreign keys are used to establish relationships between tables. For example, the `route` table has foreign keys referencing the `station` table:

These keys ensure that the `origin_station_id` and `destination_station_id` columns refer to valid entries in the `station` table.

### Indexes

Indexes can be defined to improve query performance. Each index is represented by a map containing:

- `:name` - The name of the index.
- `:table` - The table where the index is applied.
- `:columns` - The list of columns that make up the index.

### Example Index Definition

This index is created on the `:number` column of the `train` table to speed up searches for train numbers.
```edn
{:indexes [{:name :train_number_idx :table :train :columns [:number]}]}
```

## Full Example Schema

Below is a complete example of a `relate.edn` file defining a schema for a train routing system:

```edn
{:host "localhost"
 :dbtype :mysql
 :dbname :railway_routing_system
 :user :root

 :tables [{:name :train
           :columns [[:id :char 36]
                     [:number :varchar 50]
                     [:type :varchar 50]
                     [:capacity :int]
                     [:status :varchar 50]] ; e.g. 'active', 'maintenance'
           :primary-key [:id]}

          {:name :route
           :columns [[:id :char 36]
                     [:name :varchar 255]
                     [:origin_station_id :char 36]
                     [:destination_station_id :char 36]
                     [:distance_km :decimal]
                     [:estimated_duration :time]]
           :primary-key [:id]
           :foreign-keys [{:column :origin_station_id :references [:station :id]}
                          {:column :destination_station_id :references [:station :id]}]}

          {:name :station
           :columns [[:id :char 36]
                     [:name :varchar 255]
                     [:city :varchar 255]
                     [:latitude :decimal]
                     [:longitude :decimal]]
           :primary-key [:id]}

          {:name :track
           :columns [[:id :char 36]
                     [:track_number :varchar 50]
                     [:status :varchar 50]  ; e.g. 'available', 'maintenance', 'occupied'
                     [:current_train_id :char 36]] ; Train currently on the track, if any
           :primary-key [:id]
           :foreign-keys [{:column :current_train_id :references [:train :id]}]}

          {:name :schedule
           :columns [[:id :char 36]
                     [:train_id :char 36]
                     [:route_id :char 36]
                     [:departure_time :datetime]
                     [:arrival_time :datetime]
                     [:estimated_duration :time]] ; Estimated time to complete route
           :primary-key [:id]
           :foreign-keys [{:column :train_id :references [:train :id]}
                          {:column :route_id :references [:route :id]}]}

          {:name :live_status
           :columns [[:id :char 36]
                     [:train_id :char 36]
                     [:current_station_id :char 36]
                     [:next_station_id :char 36]
                     [:current_speed :decimal]
                     [:delayed_by_minutes :int] ; Real-time delay
                     [:status :varchar 50]] ; e.g. 'on-time', 'delayed', 'emergency'
           :primary-key [:id]
           :foreign-keys [{:column :train_id :references [:train :id]}
                          {:column :current_station_id :references [:station :id]}
                          {:column :next_station_id :references [:station :id]}]}]

 :indexes [{:name :train_number_idx :table :train :columns [:number]}
           {:name :route_station_idx :table :route :columns [:origin_station_id :destination_station_id]}
           {:name :schedule_train_idx :table :schedule :columns [:train_id]}
           {:name :live_status_train_idx :table :live_status :columns [:train_id]}
           {:name :track_status_idx :table :track :columns [:status]}]}
```
