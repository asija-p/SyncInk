wFront:
npm i
ng serve



Backend:
dotnet watch run



Ako nastane greska sa postgres-om uraditi sledece:

dotnet clean
dotnet restore
dotnet ef database update 0
dotnet ef migrations remove
dotnet ef migrations add v1
dotnet ef database update




Komande za Kasandru:

docker pull cassandra:latest
docker run -d --name cassandra-SyncInk -p 9042:9042 cassandra:latest
docker exec -it cassandra-SyncInk cqlsh
CREATE KEYSPACE IF NOT EXISTS syncinkcdb
WITH replication = {'class': 'SimpleStrategy', 'replication_factor': 1};

USE syncinkcdb;

CREATE TABLE IF NOT EXISTS latest_snapshot_by_user (
    user_id uuid,
    saved_at timestamp,
    room_name text,
    save_id uuid,
    PRIMARY KEY (user_id, saved_at, room_name)
) WITH CLUSTERING ORDER BY (saved_at DESC);

CREATE TABLE strokes_by_snapshot (
    user_id uuid,
    room_name text,
    save_id uuid,
    stroke_id uuid,
    color text,
    points_json text,
    saved_at timestamp,
    size double,
    stroke_date timestamp,
    visible boolean,
    PRIMARY KEY ((user_id, room_name, save_id), stroke_id)
) WITH CLUSTERING ORDER BY (stroke_id ASC);


CREATE TABLE drawing_activity_counters (
                   room_name text,
                   minute_bucket timestamp,
                   strokes_completed counter,
                   undos counter,
                   redos counter,
                   PRIMARY KEY ((room_name), minute_bucket)
               ) WITH CLUSTERING ORDER BY (minute_bucket ASC);

CREATE TABLE drawing_activity_state (
                   room_name text,
                   minute_bucket timestamp,
                   active_users set<uuid>,
                   PRIMARY KEY ((room_name), minute_bucket)
               ) WITH CLUSTERING ORDER BY (minute_bucket ASC);

Komande za Redis:

docker pull redis
docker run -d -p 6379:6379 --name redis-SyncInk -v redis-data:/data -e REDIS_APPENDONLY=yes redis





.env

ALLOWED_HOSTS=*

CONNECTIONSTRINGS__DEFAULTCONNECTION=Host=localhost;Port=5432;Database=SyncInkDB;Username=postgres;Password=vasasifra

JWT__KEY=secretkey
JWT__ISSUER=SyncInkBackend
JWT__AUDIENCE=SyncInkClient
JWT__EXPIREMINUTES=60


Napomena: Nakon registracije potrebno je uraditi login. U slucaju da SignalR ne uspostavi konekciju odmah potrebno je refresovati stranicu.
