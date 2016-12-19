# fc2016_ht4_mongodb
Hometask: 

1) Create a dump of DB as a archive in .gz format.

2) Design: Create a Posts DB with instances: authors/articles/comments/tags and others. Try to practice CRUD operation.

3) Make your DB as faster as need but not more (create Indexes).

4) There are documents for each student (student_id) across a variety of classes (class_id). Your task is to calculate the class with the best average student performance. This involves calculating an average for each student in each class of all non-quiz assessments and then averaging those numbers to get a class average.

5) Replication for DB above (point 4).

# Point 1
```
mongodump --archive=articles.gz --gzip --db articles
```
# Point 2

# Point 3

# Point 4

```
db.grades.aggregate([  
    { "$unwind": "$scores" },
    { "$match": { "scores.type": { "$ne": "quiz" } } },
    { "$group": {
        "_id": { "student_id": "$student_id","class_id":"$class_id"},        
        "student_avg_score": {
          "$avg": "$scores.score"
        }
      }
    },
    { "$group": {
        "_id": { "class_id": "$_id.class_id" },
        "class_avg_score": {
          "$avg": "$student_avg_score"          
        }
      }
    },
    { "$sort": { "class_avg_score":-1 } }, 
    { "$limit": 1 },
    { "$project": {
        "_id" : 0,
        "class_id" : "$_id.class_id",
        "avg_score" : "$class_avg_score"
      } 
    }
  ],
  {
    //explain: true
  })
```

# Point 5

Make directories for our databases
```
mkdir /Users/UserName/db1
mkdir /Users/UserName/db2
mkdir /Users/UserName/db3
```

First member
```
mongod --dbpath /Users/Username/db1 --port 27017 --replSet mainReplicaSet
```
Second member
```
mongod --dbpath /Users/Username/db2 --port 27018 --replSet mainReplicaSet
```
Third member
```
mongod --dbpath /Users/Username/db3 --port 27019 --replSet mainReplicaSet
```

Set Primary server, and start it
```
mongo --port 27017
rs.initiate()
rsconf={"_id":"mainReplicaSet","members":[{"_id":0,host:"localhost:27017"}]}
rs.reconfig(rsconf)
```

Add secondaries
```
rs.add("localhost:27018")
rs.add("localhost:27019")
```
