TRANSACTION STORAGE
=======================

Rest system for storing and analyzing payment transactions from web-based payment portals.

CONFIGURE
-----------------------

Change config/configuration.yml to suite your needs.

```
localhost:
  server:
    host:                 'http://localhost:4000'           # host name
  transaction_engine:
    host:                 'localhost'                       # transaction connection
    port:                 '4000'                                      
    ssl:                  false
    open_timeout:         5
    read_timeout:         5
    secret:               c6af25e263004d959a35f7776b0679ee  # secret used for verification
```

Run server on port 4000 (depending on your configuration).

RUNNING SYSTEM
-----------------------

```
bundle install
bundle exec rake db:migrate
bundle exec rake db:seed_fu
rails s -p 4000
```


CALLING AND AUTHENTICATING SYSTEM
---------------------

Each request is composed of:

1. Parameters - depending on function
2. Timestamp - Timestamp of call
3. Signature  - hash based signature of params and timestamp


Signature is created in following manner:

1. Sort all parameters based on parameter name
2. Build string of (key,value) pairs like : key1=value1key2=value2...
3. Append secret key to them 'key1=value1key2=value2...secret'
4. Generate SHA256 hash from that string

Example of call (create user account):


```
POST /api_account user_id=86539

signature = SHA256("ts=17376648367user_id=86539#{Config::Configuration.get(:transaction_engine, :secret)}")

POST /api_account ts=17376648367&user_id=86539&hash=#{signature}
```


!!!NOTE: NEVER SEND SECRET IN REQUEST PARAMETERS!!!


INTERFACE
---------------------

Following methods are implemented:

```
freeze_api_account POST /api/accounts/:id/freeze(.:format)

Freeze user account - only read operation can be executed on that account.

Params:
	id - account id
	
Response:
	Empty document
```

```
unfreeze_api_account POST /api/accounts/:id/unfreeze(.:format)

Unfreeze user account 

Params:
	id - account id
	
Response:
	Empty document
```

```
payment_api_account POST /api/accounts/:id/payment(.:format)

Make payment to account:

Params: 
[{
   source    : {
     name : 'Paypal',
     data : 'HFKLF97856OPYTWMX86FFKJF876'
   },
   orders : [
     {
       order : 1234,
       pieces : [
         { amount  :-120, account : 10, type : 'P' },
         { amount  :  80, account : 2, type : 'T' },
         { amount  :  15, account : 3, type : 'T' }
       ]
     }
   ]
}]

Response:
	Empty document
```

```
history_api_account GET /api/accounts/:id/history(.:format)

Get account history

Params:
	id - account id
	offset - paging offset
	
Response:
	Array of serialized transaction models belonging to that account

```


```
stats_api_account GET /api/accounts/:id/stats(.:format)

Get account statistics

Params:
	id - account id
	
Returns:
	Available balance
```


```
group_info_api_account GET /api/accounts/:id/group_info(.:format)

Get all transactions from specified group

Params:
	id -  account id
	link_id - id of group/link

Response:
	Array of serialized transaction models belonging to that group

```

```
api_accounts GET /api/accounts(.:format)

Not implemented!!!
```

```
create_account POST /api/accounts(.:format)

Create new account

Params:
	user_id - unique id from client system. 
Response:
	Serialized account model.
```

```
api_account GET /api/accounts/:id(.:format)

Get account information

Params:
	id - account id
	
Response:
	Serialized account model

```

```
update_account PUT /api/accounts/:id(.:format)

Not implemented!!!
```


```
delete_account DELETE /api/accounts/:id(.:format)

Delete account (soft delete)

Params:
	id - account id to delete
	
Response:
	Empty document
```
