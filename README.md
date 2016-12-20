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

Connect to mongo
```
mongo --host localhost --port 27017
```

Use articles database (or create if not exist)

```
use articles
```

Create db collection

```
db.createCollection("articles",{
  validator:{
    $or:[      
      {comments:{$type:"array"}}, 
      {tags:{$type:"array"}},
      {type:{$type:"string"}},
      {authors:{$type:"array"}},
      {dateCreated: {$type: "date"}},
      {text: {$type: "string"}},
      {name: {$type: "string"}},
      {title: {$type: "string"}}
    ]
  }
})
```

insert authirs:
```
db.articles.insertMany([{_id:"superauthor@gmail.com", name:"Jon Smith", type: "author"},{_id:"nelson@tut.by", name:"Nelson Winter", type: "author"},{_id:"skankhunt42@gmail.com", name: "Gerald Broflovski", type: "author"}])
```
insert categories:
```
db.articles.insertMany([{_id:"it-cat", name: "IT", type: "category"}, {_id:"animals-cat", name: "animals", type: "category"}, {_id:"web-cat", name: "Web", type: "category"}, , {_id:"policy-cat", name: "Policy", type: "category"}])
```
insert articles:
```
db.articles.insertOne({title:"best practices of HTML5", tags:["html","frontend", "web"],dateCreated: new Date(), text:"HTML5 and bla bla bla", comments:[{text:"Nice!"},{text: "Thank you."}],authors:["superauthor@gmail.com"],categories:["web-cat"], type: "article"})
db.articles.insertOne({title:"Some article", tags:["six","five", "four"],dateCreated: new Date(), text:"Pshhhshshshsh", comments:[],authors:["nelson@tut.by"],categories:["policy-cat"], type: "article"})
db.articles.insertOne({title:"Cat - developer in Amazon", tags:["cat", "crazy"],dateCreated: new Date(), text:"Cat - developer in Amazon", comments:[{text:"Wooooww!"},{text: "Thank you."}, {text: "I find you"}],authors:["skankhunt42@gmail.com"],categories:["animals-cat", "policy-cat"], type: "article"})
db.articles.insertOne({title:"React and friends", tags:["html","frontend", "web"],dateCreated: new Date(), text:"React + Redux + Less", comments:[{text: "Thank you."}],authors:["superauthor@gmail.com"],categories:["web-cat"], type: "article"})
```
updating example:
```
db.articles.updateOne(
   {_id: "superauthor@gmail.com"},
   { $set: { "name": "John Doe"} }
)
```
deleting example:
```
 db.articles.deleteOne( { "_id" : "it-cat" } )
```
finding example (find articles by tag) :
```
db.articles.find({ tags: {$in: ["html", "web"]} })
```
# Point 3

Create indexes:
```
db.articles.createIndex( { type: 1 } )
db.articles.createIndex({"dateCreated" : -1})
db.articles.createIndex( { text: "text", tags: "text" } )
```
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
