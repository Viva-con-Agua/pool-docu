# HowTo handle request with [axios](https://github.com/axios/axios)

## Multiple Request

In a microservice system, multiple requests for a data model cannot always be avoided.
A "good" example of this is the signin process of the frontend apps.

We request drops for the user model and have to request stream for the same reason. 

```
  // drops identity request
  function getDropsIdentity() {
    return axios.get('/drops/webapp/identity');
  }

  // stream idenity request
  function getStreamIdentity() {
    return axios.get('/backend/stream/identity');
  }
```

The `getDropsIdentity` function returns the result of the drops request and the `getStreamIdentity` function do the same for stream.

Now we can synchronize the requests via axios.all([]) which gives us the opportunity to handle a list of "request functions". 

```
   axios.all([getDropsIdentity(), getStreamIdentity()])
     // handle responses
     .then(axios.spread((drops, stream) => {
       stream.data["name"] = drops.data.additional_information.profiles[0].supporter.fullName
       store.dispatch('user/success', stream.data)
       checker()
     }))
     // handle errors
     .catch((error) => {
       store.dispatch('user/error', error)
     })
```

The `.then` block will only execute if both requests are successful. So we can add the `name` and store it into `vuex` 

