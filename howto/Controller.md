# Api Contoller

## Post Model Controller
Controller for creating models

### Request
 `POST \model`

### Controller
```
func Post(c echo.Context) (err error) {

	// first create a new model for your body
	body := new(models.Model)

	// save data to body
	if err = c.Bind(body); err != nil {
		return c.JSON(http.StatusBadRequest, err)
	}

	// validate body
	if err = c.Validate(body); err != nil {
		return c.JSON(http.StatusBadRequest, err)
	}

	// insert body into database by using the database controller
	if err = database.Insert(body); err != nil {
    
    // if the model exist in database return 409 with Confict response
		if strings.Contains(err.Error(), "Duplicate entry") {
			return c.JSON(http.StatusConflict, models.Conflict())
		}

    // all other error will log by database controller and return 500 InternelServerError
		return c.JSON(http.StatusInternalServerError, models.InternelServerError())	}
	
  // response created
	return c.JSON(http.StatusCreated, models.Created())
}
```

## Get By Id

### Request
 `GET \model\:id`

### Controller
```
func GetById(c echo.Context) (err error) {
  
  // get id from url
	uuid := c.Param("id")

	// get model form database
  response, err := database.SelectById(uuid)
	if err != nil {
    
    // in case of database error return InternelServerError 
		return c.JSON(http.StatusInternalServerError, models.InternelServerError)
	}
  // in case there isn't a model return NoContent
	if response == nil {
		return c.JSON(http.StatusNoContent, models.NoContent(uuid))
	}
  // return model
	return c.JSON(http.StatusOK, response[0])
}
```

## Get List 

### Request
 `GET \model?query`


### Controller
```
func GetList(c echo.Context) (err error) {
  // select query
	query := new(models.Query)
	if err = c.Bind(query); err != nil {
		return c.JSON(http.StatusInternalServerError, err)
	}
  
  // create Page object from query
	page := query.Page()
  
  // create Sort string from query
	sort := query.OrderBy()
  
  // create filter from query
	filter := query.Filter()
	
  // call database
  response, err := database.SelectList(page, sort, filter)

  // in case error there is an error return InternelServerError
	if err != nil {
		return c.JSON(http.StatusInternalServerError, models.InternelServerError)
	}

  // return List
	return c.JSON(http.StatusOK, response)
}
```

## Update 

### Request
  `PUT \model`

### Controller
```
func Update(c echo.Context) (err error) {
	
  // create model for body 
	body := new(models.User)
	
  // save data to body
	if err = c.Bind(body); err != nil {
		return c.JSON(http.StatusBadRequest, err)
	}
	// validate body
	if err = c.Validate(body); err != nil {
		return c.JSON(http.StatusBadRequest, err)
	}
	// update body into database
	if err = database.UpdateUser(body); err != nil {
		if err == utils.ErrorNotFound {
			return c.JSON(http.StatusNoContent, models.NoContent(body.Uuid))
		}
		return c.JSON(http.StatusInternalServerError, models.InternelServerError())
	}
	// response created
	return c.JSON(http.StatusOK, models.Updated(body.Uuid))
}
```

## Delete 

### Request
  `DELETE \model`

### Controller
```
func DeleteUser(c echo.Context) (err error) {

  // create body as models.User
	body := new(models.DeleteBody)

  // save data to body
	if err = c.Bind(body); err != nil {
		return c.JSON(http.StatusBadRequest, err)
	}

	// validate body
	if err = c.Validate(body); err != nil {
		return c.JSON(http.StatusBadRequest, err)
	}

	// update body into database
	if err = database.DeleteUser(body); err != nil {
		if err == utils.ErrorNotFound {
			return c.JSON(http.StatusNoContent, models.NoContent(body.Uuid))
		}
		return c.JSON(http.StatusInternalServerError, models.InternelServerError())
	}

	// response created
	return c.JSON(http.StatusOK, models.Deleted(body.Uuid))
}
```

## Join two models

### Request 
 `POST \model1\model2`

### Controller

```
func JoinModel1Model2(c echo.Context) (err error) {

	// create body as models.Role
	body := new(models.AssignBody)

	// save data to body
	if err = c.Bind(body); err != nil {
		return c.JSON(http.StatusBadRequest, err)
	}

	// validate body
	if err = c.Validate(body); err != nil {
		return c.JSON(http.StatusBadRequest, err)
	}

	// insert body into database
	if err = database.JoinUserRole(body); err != nil {
		return c.JSON(http.StatusInternalServerError, models.InternelServerError)
	}

	// response created
	return c.JSON(http.StatusCreated, models.Created())
}
```
