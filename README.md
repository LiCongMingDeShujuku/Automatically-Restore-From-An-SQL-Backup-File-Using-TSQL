![CLEVER DATA GIT REPO](https://raw.githubusercontent.com/LiCongMingDeShujuku/git-resources/master/0-clever-data-github.png "李聪明的数据库")

# 使用TSQL从SQL备份文件自动还原
#### Automatically Restore From An SQL Backup File Using TSQL
**发布-日期: 2014年04月08日 (评论)**


## Contents

- [中文](#中文)
- [English](#English)
- [SQL Logic](#Logic)
- [Build Info](#Build-Info)
- [Author](#Author)
- [License](#License) 


## 中文
你有大量SQL数据库以及大量备份文件需要还原。如果你像我一样宁愿为所有东西编码，那你只需编写逻辑，就可以让事情变得更容易。我写了一些可能对你有帮助的恢复逻辑，你只需要提供数据库名称和备份文件的位置就可以了。

那么具体如何操作呢？

它会获取你拥有的备份文件，并从文件中推断出Logical和Physical名称，然后恢复。恢复具有以下内容（使用替换和恢复），因此它肯定会覆盖它正在恢复的现有数据库。

需要：
Sp_KillAllProcessInDB
可以在以下网页找到相关信息：http://techresource.appvui.biz/1547686/ms-sql-server-kill-processes-particular-db.html

---

## English
You have a bunch of SQL Database Restores you have to do, and a ton of backup files.  If you’re like me you would prefer to code everything out.  Just write the logic, and make things alot easier.  I wrote some restore logic that might help you.   All you need to do is provide the database name, and the location to backup file.

Ok so what does this do exactly?

It takes the backup file you have, and extrapolates the Logical, and Physical name from the file, and passes it into a restore.   The restore has the following ( with replace, recovery ) so it will definitely overwrite the existing database that it’s restoring too.

Dependencies:
Sp_KillAllProcessInDB
Can be found here:  http://techresource.appvui.biz/1547686/ms-sql-server-kill-processes-particular-db.html


---
## Logic
```SQL
use master;
set nocount on
declare @database_name varchar(255)
declare @backup_file_name varchar(255)
set @database_name = ‘MyDatabaseNameHere’
set @backup_file_name = ‘MyPathToBackupFileHere’
declare @filelistonly table
(
	logicalname 		nvarchar(128)
, 	physicalname 		nvarchar(260)
, 	[type] 				char(1)
, 	filegroupname 		nvarchar(128)
, 	size 				numeric(20,0)
, 	maxsize 			numeric(20,0)
, 	fileid 				bigint
, 	createlsn 			numeric(25,0)
, 	droplsn 			numeric(25,0)
, 	uniqueid 			uniqueidentifier
, 	readonlylsn 		numeric(25,0)
, 	readwritelsn 		numeric(25,0)
, 	backupsizeinbytes 	bigint
, 	sourceblocksize 	int
, 	filegroupid 		int
, 	loggroupguid 		uniqueidentifier
, 	differentialbaselsn	numeric(25,0)
, 	differentialbaseguid	uniqueidentifier
, 	isreadonl 			bit
, 	ispresent 			bit
, 	tdethumbprint 		varbinary(32)
)
insert into
@filelistonly exec (‘restore filelistonly from disk = ”’ + @backup_file_name + ””)
declare @restore_line0 varchar (255)
declare @restore_line1 varchar (255)
declare @restore_line2 varchar (255)
declare @stats varchar (255)
declare @move_files varchar (max)
set 	@restore_line0 = (‘use master;’)
set 	@restore_line1 = (‘exec master..sp_killallprocessindb ”’ + @database_name + ”’;’)
set 	@restore_line2 = (select ‘restore database [‘ + @database_name + ‘] from disk = ”’ + @backup_file_name + ”’ with replace, recovery, ‘)
set 	@stats = (‘stats = 20;’)
set 	@move_files = ”
select 	@move_files = @move_files + ‘move ”’ + logicalname + ”’ to ”’ + physicalname + ”’,’ + char(10) from @filelistonly order by fileid asc
exec
(
	@restore_line0
+ 	@restore_line1
+ 	@restore_line2
+ 	@move_files
+ 	@stats
)

```

[![WorksEveryTime](https://forthebadge.com/images/badges/60-percent-of-the-time-works-every-time.svg)](https://shitday.de/)

## Build-Info

| Build Quality | Build History |
|--|--|
|<table><tr><td>[![Build-Status](https://ci.appveyor.com/api/projects/status/pjxh5g91jpbh7t84?svg?style=flat-square)](#)</td></tr><tr><td>[![Coverage](https://coveralls.io/repos/github/tygerbytes/ResourceFitness/badge.svg?style=flat-square)](#)</td></tr><tr><td>[![Nuget](https://img.shields.io/nuget/v/TW.Resfit.Core.svg?style=flat-square)](#)</td></tr></table>|<table><tr><td>[![Build history](https://buildstats.info/appveyor/chart/tygerbytes/resourcefitness)](#)</td></tr></table>|

## Author

- **李聪明的数据库 Lee's Clever Data**
- **Mike的数据库宝典 Mikes Database Collection**
- **李聪明的数据库** "Lee Songming"

[![Gist](https://img.shields.io/badge/Gist-李聪明的数据库-<COLOR>.svg)](https://gist.github.com/congmingshuju)
[![Twitter](https://img.shields.io/badge/Twitter-mike的数据库宝典-<COLOR>.svg)](https://twitter.com/mikesdatawork?lang=en)
[![Wordpress](https://img.shields.io/badge/Wordpress-mike的数据库宝典-<COLOR>.svg)](https://mikesdatawork.wordpress.com/)

---
## License
[![LicenseCCSA](https://img.shields.io/badge/License-CreativeCommonsSA-<COLOR>.svg)](https://creativecommons.org/share-your-work/licensing-types-examples/)

![Lee Songming](https://raw.githubusercontent.com/LiCongMingDeShujuku/git-resources/master/1-clever-data-github.png "李聪明的数据库")

