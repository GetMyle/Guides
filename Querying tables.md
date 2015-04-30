#### Querying tables ####

Start with the following: `var Query = require("mylet-query");`. This brings a `Query` object that allows to do queries.

Queries always start with the following statement: `new Query("master");`. That means that master table (a table that has all records) is going to be requested to get records from.

Then we have to specify what fields we want to select from `master` table: `new Query("master").select("phrase");`. Available fields to select:

- `time` - UTC time of when a record was added

- `phrase` - record phrase

- `latitude`

- `longitude`

- `altitude`

To specify more than one field, use "comma" to separate them: `.select("latitude", "longitude")`.

Next step is to specify records filter: `new Query("master").select("phrase").where({ phrase: { $contains: "water"} });`. The same fields as above are used to create `where` clause.

Here is list of supported group operators:

 - `$and` - AND. Examples: `{ latitude: 4, longitude: 6 }` is equal to `{ $and: { latitude: 4, longitude: 6 } }` and is equal to `{ $and: [ { latitude: 4 }, { longitude: 6 } ] }` and means latitude equals 4 and longitude equals 6

 - `$or` - OR. Examples: `{ $or: { latitude: 4, longitude: 6 } }` is equal to `{ $or: [ { latitude: 4 }, { longitude: 6 } ] }` and means latitude equal 4 or longitude equals 6

 - `$not` - negation. Examples: `{ $not: { latitude: 4 } }` means latitude not equal to 4; `{ latitude: 4, { $not: { longitude: 6 } } }` means latitude equals 4 and longitude not equals to 6

 Here is list of supported field operators:

- `$e` - equals. `{ latitude: { $e: 4 } }` means latitude equals 4 and is the same as `{ latitude: 4 }`

- `$ne` - no equal to. `{ latitude: { $ne: 4 } }` means latitude not equal to 4

- `$gte` - greater than or equal to. `{ latitude: { $gte: 4 } }` means latitude is greater than or equal to 4

- `$gt` - greater than. `{ latitude: { $gte: 4 } }` means latitude is greater than 4

- `$lte` - less than or equal to. `{ latitude: { $lte: 4 } }` means latitude is less than or equal to 4

- `$lt` - less than. `{ latitude: { $lt: 4 } }` means latitude is less than 4

- `$contains` - string contains. `{ phrase: { $contains: "test" } }` means phrase contains "test" string. Applicable only to `phrase` field

- `$startsWith` - string starts with. `{ phrase: { $startsWith: "a" } }` means phrase starts with "a" string. Applicable only to `phrase` field

- `$endsWith` - string ends with. `{ phrase: { $endsWith: "a" } }` means phrase ends with "a" string. Applicable only to `phrase` field

- `$in` - is in array. `{ altitude: { $in: [1, 2, 3, 4] } }` means altitude has value 1, 2, 3 or 4

Once query is built there is need to execute it. `toArray()` method does it:

 ```js
 new Query("master")
 	.select("phrase")
 	.where({
 		phrase: { $contains: "water"}
 	})
 	.toArray(function(err, data) {
 		// handle data here
 	});
 ```

`toArray()` method accepts a callback function that is called once result-set is ready. It has two parameters:

- `err` - error object in case of an error

- `data` - result-set with `fields` and `rows` properties 


#### Joining tables ####

To join tables you have to specify them seprated by comma in `Query` constructor:

```js
new Query("master", "food")
```

Fields to select have to be specified with table name prefix to avoid conflicts:

```js
new Query("master", "food")
    .select("master.time", "food.calories")
``` 

Join criteria is specified in `where` part:

```js
new Query("master", "food")
    .select("master.time", "food.calories")
    .where({
      "master.phrase": {
          $contains: {
              $valueOf: "food.name"
          }
    })
```

Note `$valueOf` field is used to specify another party of table join.