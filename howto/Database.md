# Database Controller Go

This guide is a collection of different examples for creating a database controller with Go for the pool².


## Select Query

### Simple Select

## Datatypes using in Pool²

In some cases it makes sense to initialize variables in the backend.
For example, the create date should not be initialize in the frontend where it could be manipulated.
A good solution is to handle these variables in the database controllers.
The Pool² uses not primitive data types such as UUID for system-wide identifiers in various cases. 
For this you will find a summary in this section of how you can create these data types.

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
In case of an error, return it and output the error via `log.Print()`. 
```
	tx, err := utils.DB.Begin()
	if err != nil {
		log.Print("Database Error: ", err)
		return err
```
The next step is to define the query. That happens in plain SQL. The ? can then be filled with variables.
```
	query := "INSERT INTO Model (value_1, value_2, ...) VALUES(?, ?, ...)" 
```
This query can now be executed via `tx.Exec()`. 
If the database throws an error, a `tx.Rollback()` should be carried out.
At the end we return the commit. If an error has occurred, the return value is (nil, err) where err! = nil
```
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
The model is saved in two tables with a foreign key. The controller first creates the main model and then gets the ID via `res.LastInsertId`. So we can simple insert the child Model and commit the request.
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


### Models with two foreign key
The controller creates a model with two foreign keys. For this we get the ids of the parents with two requests.
Then the model can be created with the keys. For more information about select query, look at (insert link)
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
