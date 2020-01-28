![MIKES DATA WORK GIT REPO](https://raw.githubusercontent.com/mikesdatawork/images/master/git_mikes_data_work_banner_01.png "Mikes Data Work")        

# Use SQL To Track Robocopy Duration 
**Post Date: November 27, 2015**        



## Contents    
- [About Process](##About-Process)  
- [SQL Logic](#SQL-Logic)  
- [Build Info](#Build-Info)  
- [Author](#Author)  
- [License](#License)       

## About-Process

<p>Here's some SQL logic that will track Robocopy Durations. I created this for an ETL process that basically takes a local database backup and copies it to another database server. I then collect the duration in a temp table for reporting of the file copy.</p>      


## SQL-Logic
```SQL
use master;
set nocount on
 
if object_id('tempdb..#robocopy_duration') is not null
    drop table #robocopy_duration
 
create table #robocopy_duration
(
    time_start      datetime
,   time_finish     datetime
,   notes           varchar(max)
)
 
declare @time_start datetime = ( select getdate())
insert into #robocopy_duration ([time_start])
values (@time_start)
 
exec master..xp_cmdshell 'ROBOCOPY "E:\LOAD_ETLS_SOURCE_BACKUPS"  "\\MyDestinationServer\E$\LOAD_ETLS_SOURCE_BACKUPS" LOAD_ETLS_MyDatabase_01.BAK /ETA /Z /XO /R:2 /W:3 /IS /B /COPYALL /NP /LOG:"E:\LOAD_ETLS_SOURCE_BACKUPS\robocopy_log_for_LOAD_ETLS_MyDatabase_01.log"'
exec master..xp_cmdshell 'ROBOCOPY "E:\LOAD_ETLS_SOURCE_BACKUPS"  "\\MyDestinationServer\E$\LOAD_ETLS_SOURCE_BACKUPS" LOAD_ETLS_MyDatabase_02.BAK /ETA /Z /XO /R:2 /W:3 /IS /B /COPYALL /NP /LOG:"E:\LOAD_ETLS_SOURCE_BACKUPS\robocopy_log_for_LOAD_ETLS_MyDatabase_02.log"'
 
declare @begin_time     datetime = ( select max([time_start]) from #robocopy_duration )
declare @time_finish    datetime = ( select getdate() )
update  #robocopy_duration
set     [time_finish]   = @time_finish where [time_start] = @begin_time
 
declare         @get_robocopy_log   table (robocopy_output varchar(max))
insert into     @get_robocopy_log select * from openrowset(bulk N'e:\load_etls_source_backups\robocopy_log_for_load_etls_MyDatabase_01.log', single_blob) as grl
 
update #robocopy_duration
    set notes = ( select robocopy_output from @get_robocopy_log )
    where [time_finish] = @time_finish
 
select
    'time_start'    = left([time_start], 19) + ' ' + datename(dw, [time_start])
,   'time_finish'   = left([time_finish], 19) + ' ' + datename(dw, [time_finish])
,   'duration'      =
         cast(datediff(second, [time_start], [time_finish])/60/60%24 as nvarchar(50)) + ' hr ' +
         cast(datediff(second, [time_start], [time_finish])/60%60 as nvarchar(50)) + ' mn ' + 
         cast(datediff(second, [time_start], [time_finish])%60 as nvarchar(50)) + 's'
,   'notes'         = notes
from
    #robocopy_duration
order by
    [time_start] asc
```

<p>Of course you could always query the Log output file directly from within SQL Server with this:</p>      


## SQL-Logic
```SQL
declare         @get_robocopy_log   table (robocopy_output varchar(max))
insert into     @get_robocopy_log select * from openrowset(bulk N'e:\load_etls_source_backups\robocopy_log_for_load_etls_MyDatabase_01.log', single_blob) as the_robocopy_log
 
select robocopy_output from @get_robocopy_log
```

This could be edited further using bulk insert and the appropriate switches to collect only the text you want during the import. 


[![WorksEveryTime](https://forthebadge.com/images/badges/60-percent-of-the-time-works-every-time.svg)](https://shitday.de/)

## Build-Info

| Build Quality | Build History |
|--|--|
|<table><tr><td>[![Build-Status](https://ci.appveyor.com/api/projects/status/pjxh5g91jpbh7t84?svg?style=flat-square)](#)</td></tr><tr><td>[![Coverage](https://coveralls.io/repos/github/tygerbytes/ResourceFitness/badge.svg?style=flat-square)](#)</td></tr><tr><td>[![Nuget](https://img.shields.io/nuget/v/TW.Resfit.Core.svg?style=flat-square)](#)</td></tr></table>|<table><tr><td>[![Build history](https://buildstats.info/appveyor/chart/tygerbytes/resourcefitness)](#)</td></tr></table>|

## Author

[![Gist](https://img.shields.io/badge/Gist-MikesDataWork-<COLOR>.svg)](https://gist.github.com/mikesdatawork)
[![Twitter](https://img.shields.io/badge/Twitter-MikesDataWork-<COLOR>.svg)](https://twitter.com/mikesdatawork)
[![Wordpress](https://img.shields.io/badge/Wordpress-MikesDataWork-<COLOR>.svg)](https://mikesdatawork.wordpress.com/)

     
## License
[![LicenseCCSA](https://img.shields.io/badge/License-CreativeCommonsSA-<COLOR>.svg)](https://creativecommons.org/share-your-work/licensing-types-examples/)

![Mikes Data Work](https://raw.githubusercontent.com/mikesdatawork/images/master/git_mikes_data_work_banner_02.png "Mikes Data Work")

