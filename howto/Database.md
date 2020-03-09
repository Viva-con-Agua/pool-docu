# Database Controller


## Datatypes
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
### simple Model

Insert simple Models into Database

```
/**
 * insert Model into database
 */
func InsertModel(r *models.ModelCreate) (err error) {
	
	// Create uuid
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
	rows2, err := utils.DB.Query("SELECT id FROM Model_2 WHERE uuid = ?", assign.RoleUuid)
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
