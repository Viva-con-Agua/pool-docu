# Database Controller Go

This guide is a collection of different examples for creating a database controller with Go for the pool².
We use the [Go-MySQL-Driver](https://github.com/go-sql-driver/mysql) and [MySQL 8.0](https://dev.mysql.com/doc/refman/8.0/en/) deployed in [Docker](https://www.docker.com/). If you want to deploy your own docker use as default [mysql-docker](https://hub.docker.com/_/mysql).


## Datatypes used in Pool²

In some cases it makes sense to initialize variables in the backend.
For example, the create date should not be initialized in the frontend, where it could be manipulated.
A good solution is to handle these variables in the database controllers.
The Pool² uses non primitive data types such as UUID for system-wide identifiers in various cases. 
Following, you will find a summary of how you can create these data types with Go.

### UUID
```
  Uuid, err := uuid.NewRandom()
	if err != nil {
		log.Print("Database Error: ", err)
		return err
	}
```
### Date
```
 time.Now().Unix()
```
## Insert Models
This section contains a summary of various cases that can be encountered when writing an insert controller.

### simple Model

Insert simple Models into Database.

With insert, it makes sense to generate a query via `DB.begin()`. 
In case of an error, return error and print the error via `log.Print()`. 
```
	tx, err := utils.DB.Begin()
	if err != nil {
		log.Print("Database Error: ", err)
		return err
```
The next step is to define the query. That happens in plain SQL. 
You can use the ? as a spaceholder for variables.
```
	query := "INSERT INTO Model (value_1, value_2, ...) VALUES(?, ?, ...)" 
```
This query can now be executed via `tx.Exec()`. 
If the database throws an error, a `tx.Rollback()` should be carried out.
At the end we return the commit. If an error has occurred, the return value is (nil, err) where err! = nil
```

	_, err = tx.Exec( query, Value_1, Value_2, ...)
	if err != nil {
		tx.Rollback()
		log.Print("Database Error: ", err)
		return err
	}
	return tx.Commit()

```

#### Full Controller
```
/**
 * insert Model into database
 */
func InsertModel(r *models.ModelCreate) (err error) {
	// begin database query and handle error
	tx, err := utils.DB.Begin()
	if err != nil {
		log.Print("Database Error: ", err)
		return err
	}
		
	query := "INSERT INTO Model (value_1, value_2, ...) VALUES(?, ?, ...)" 
	// insert role
	_, err = tx.Exec( query, Value_1, Value_2, ...)

	if err != nil {
		tx.Rollback()
		log.Print("Database Error: ", err)
		return err
	}
	return tx.Commit()
}


```
### Model with foreign key
The model is saved in two tables with a foreign key. The controller first creates the main model and then gets the ID via `res.LastInsertId`. So we can simply insert the child Model and commit the request.
```
	//insert Model
	res, err := tx.Exec("INSERT INTO Model (uuid, updated, created) VALUES(?, ?, ?)", Uuid.String(), time.Now().Unix(), time.Now().Unix())
	if err != nil {
		tx.Rollback()
		log.Print("Database Error: ", err)
		return err
	}
	// get user id via LastInsertId
	id, err := res.LastInsertId()
	if err != nil {
		log.Print("Database Error: ", err)
		return err
	}

```
#### Full Controller

```
func InsertWithKey(c *Model) (err error) {
	
	// Create uuid
	Uuid, err := uuid.NewRandom()
	if err != nil {
		log.Print("Database Error: ", err)
		return err
	}

	// begin database query and handle error
	tx, err := utils.DB.Begin()
	if err != nil {
		log.Print("Database Error: ", err)
		return err
	}

	//insert Model
	res, err := tx.Exec("INSERT INTO Model (uuid, updated, created) VALUES(?, ?, ?)", Uuid.String(), time.Now().Unix(), time.Now().Unix())
	if err != nil {
		tx.Rollback()
		log.Print("Database Error: ", err)
		return err
	}
	// get user id via LastInsertId
	id, err := res.LastInsertId()
	if err != nil {
		log.Print("Database Error: ", err)
		return err
	}
	
	// insert credentials
	res, err = tx.Exec("INSERT INTO Model_2 (value_1, value_2, Model_id) VALUES(?, ?, ?)", value_1, value_2, id)
	if err != nil {
		tx.Rollback()
		log.Print("Database Error: ", err)
		return err
	}
	// insert profile
	return tx.Commit()
}
```


### Models with two foreign keys
The controller creates a model with two foreign keys. 
We receive the ID's of the parent Models with two requests.
```
  // select model_1 from database
	rows, err := utils.DB.Query("SELECT id FROM Model_1 WHERE uuid = ?", assign.Assign)
	if err != nil {
		log.Print("Database Error", err)
		return err
	}
	// select id from rows
	var model_1_id int
	for rows.Next() {
		err = rows.Scan(&model_1_id)
		if err != nil {
			log.Print("Database Error: ", err)
			return err
		}
	}
  ...

```
From that, the model can be created with the keys. 
```
	res, err := tx.Exec("INSERT INTO Model (uuid, model_1_id, model_2_Id) VALUES(?, ?, ?)", Uuid, model_1_id, model_2_Id)
	if err != nil {
		tx.Rollback()
		log.Print("Database Error: ", err)
		return err
	}
	return tx.Commit()

```
For more information about select query, visit (insert link)

#### Full Controller
```
func InsertModelWithTwoKeys(assign *models.AccessUserCreate) (err error) {
	
  // select model_1 from database
	rows, err := utils.DB.Query("SELECT id FROM Model_1 WHERE uuid = ?", assign.Assign)
	if err != nil {
		log.Print("Database Error", err)
		return err
	}
	// select id from rows
	var model_1_id int
	for rows.Next() {
		err = rows.Scan(&model_1_id)
		if err != nil {
			log.Print("Database Error: ", err)
			return err
		}
	}
	// select model_2 from database
	rows2, err := utils.DB.Query("SELECT id FROM Model_2 WHERE uuid = ?", assign.To)
	if err != nil {
		log.Print("Database Error", err)
		return err
	}
	//select user_id from rows
	var model_2_Id int
	for rows2.Next() {
		err = rows2.Scan(&model_2_Id)
		if err != nil {
			log.Print("Database Error: ", err)
			return err
		}
	}
	// begin database query and handle error
	tx, err := utils.DB.Begin()
	if err != nil {
		log.Print("Database Error: ", err)
		return err
	}
	// Create uuid
	Uuid, err := uuid.NewRandom()
	if err != nil {
		log.Print("Database Error: ", err)
		return err
	}
	

	res, err := tx.Exec("INSERT INTO Model (uuid, model_1_id, model_2_Id) VALUES(?, ?, ?)", Uuid, model_1_id, model_2_Id)
	if err != nil {
		tx.Rollback()
		log.Print("Database Error: ", err)
		return err
	}
	return tx.Commit()
}
```

## Select Models from database

In this section we will go into select. 
Select functions are the most used functions in a web service. Therefore this part of the documentation should be discussed further.

## Query

### Limit the query for selecting only one element 

First, we go into a query that simply returns a model. The UUID can be easily filtered by using `WHERE m.uuid = ?`.
```
query := "SELECT m.uuid, m.name, m.service_name, m.created " + 
					 "FROM Model AS m " +
					 "WHERE r.uuid = ? " 
```
Furthermore, the UUID is also unique, so the query above returns only one mod. 
However, the database will not stop searching, but goes through all elements in the table.
So if you want select only one Element add `Limit 1`. 
The query will stop iterate and return the model.

```
query := "SELECT m.uuid, m.name, m.service_name, m.created " + 
					 "FROM Model AS m " +
					 "WHERE r.uuid = ? " +
           "LIMIT 1 "

```
### Limit and Page

You can use `LIMIT ?, ?` for paging your database select. The first ? is the element you start selecting and the second ? is the count of how many.
For example `LIMIT 0, 20` select the first 20 elements. 

### Left Join one to many relation

A big problem in on to many relation are handling duplicates in the selected table. 
The naive solution is build a parser and handle with dublicates. 
The problem is that each select query needs a seperated parser. This parser need care about sorting and debugging is very uncomfortable. 

A second solution is using `DISTINCT` and search child models with a seperated database request. 
It works fine with short lists or single elements, but it doesn't scale.

The best way to handle with on to n table dublicates is to keep all informations in a table with no dublicates.^^
Easy to parse and easy to convert. MySql provide some functions they help us out.

```
UserQuery = "SELECT u.id, u.uuid, c.email, u.updated, u.created, CONCAT('[', " +
	"GROUP_CONCAT(JSON_OBJECT('uuid', a.uuid, 'role_uuid', r.uuid, 'role_name', r.name, 'service_name', r.service_name, "	+
	"'model_uuid', m.uuid, 'model_name', m.name, 'created', a.created)), ']') " +
		"FROM User AS u " +
		"LEFT JOIN Credentials AS c ON u.id = c.User_id " +
		"LEFT JOIN AccessUser AS a ON u.id = a.User_id " +
		"LEFT JOIN Model AS m ON a.id = m.AccessUser_id " +
		"LEFT JOIN Role AS r ON a.Role_Id = r.id " 
```
The above query is for selecting a user with a list of access. We can easy build AccessUser via `JSON_OBJECT`.
In case we have more than one AccessUser `GROUP_CONCAT` will seperate all JSON_OBJECT with `,`. 
Than simple `CONCAT('[', List , ']')` and you build an Json_Array.

The result can be easily converted to Json. 
First define dummy for for row bytes. 
```
	//initial dummy varibles
	var accessByte []byte
	var id int
	
```

The `rows.Next()` return the next lines of the table. We have no duplicates, so we scan user by user.
The user can direct assign to an models.User. 

```
		// create User 
		user := new(models.User)
		//scan row and fill user
		err = rows.Scan(&id, &user.Uuid, &user.Email, &user.Updated, &user.Created, &accessByte)
    // default error handling
		if err != nil {
			log.Print("Database Error: ", err)
			return nil, err
		}

```
For handling the Json List we store it as []byte and use `json.Unmarshal` to assign the list to access. Go handle it very smart.
```
    access := new([]models.AccessUser)
		// create json from []byte
		err = json.Unmarshal(accessByte, &access)

```

Last check if the AccessUserList has an element. The query return allways a json and we assign it to a model. If a User hasn't a AccessUser, the database return the object with empty values, so the first element of the list have no uuid. 

```
		if ((*access)[0].Uuid != "") {
			user.Access = *access
		}else{
			user.Access = nil
		}

```

#### Full Controller

```
	//initial dummy varibles
	var accessByte []byte
	var id int
	
	// convert each row to User
	for rows.Next() {
		
		// create User and AccessUser
		user := new(models.User)
		access := new([]models.AccessUser)

		//scan row and fill user
		err = rows.Scan(&id, &user.Uuid, &user.Email, &user.Updated, &user.Created, &accessByte)
		if err != nil {
			log.Print("Database Error: ", err)
			return nil, err
		}
		// create json from []byte
		err = json.Unmarshal(accessByte, &access)

		if err != nil {
			log.Print("Database Error: ", err)
			return nil, err
		}
		// add roles to user
		if ((*access)[0].Uuid != "") {
			user.Access = *access
		}else{
			user.Access = nil
		}
		// append to list of user
		users = append(users, *user)
	}
	return users, err
}
```

### Select by UUID

```
func SelectModelById(uuid string) (model []Model, err error) {
	// Execute the Query
	query := "SELECT r.uuid, r.name, r.service_name, r.created " + 
					 "FROM Role AS r " +
					 "WHERE r.uuid = ? " 
	rows, err := utils.DB.Query(query, uuid)
	if err != nil {
		log.Print("Database Error", err)
		return nil, err
	}

	// convert each row
	for rows.Next() {
		
		//create new Model
		model := new(Model)

		//map row to models.Role
		err = rows.Scan(&model.Uuid, &model.Name, &model.ServiceName, &model.Created)
		if err != nil {
			log.Print("Database Error: ", err)
			return nil, err
		}
		models = append(models, *model)
	}
	return models, err
}
```

## Update model

Similar to Insert a model some values shouldn't set by frontend. 
For example UUID and created will never updated. 
The updated value is the actuall utc time and should set by backend.

### Simple update
The function below show a solution for update. As identifier we use the uuid of the current model.

```
func UpdateUser(user *models.User) (err error) {
	// sgl begin
	tx, err := utils.DB.Begin()
	if err != nil {
		log.Print("Database Error: ", err)
		return err
	}
	//update user user
	_, err = tx.Exec("UPDATE User SET updated = ? WHERE uuid = ?", time.Now().Unix(), user.Uuid)
	if err != nil {
		tx.Rollback()
		log.Print("Database Error: ", err)
		return err
	}
	return tx.Commit()

}
```



