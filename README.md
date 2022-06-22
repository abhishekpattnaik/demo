## Golang Scaffolding

This is a base code for golang describing the following broad concepts:

1. Server Setup
2. Command setup
3. Router setup
4. Embedding Middlewares
5. Config Management

## Golang Setup
brew install Go@1.18
Golang IDE > Preferences > Enable Go Modules

## Dev Setup
- Clone the repository
- Run the following commands
```
go env -w GOPRIVATE="bitbucket.org/apps-for-bharat/*"
git config --global url.ssh://git@bitbucket.org/.insteadOf https://bitbucket.org/
go env -w GO111MODULE=on
```

### Make Commands
```
make test
make lint
make build

```

### Pre Push Commands

```
make pre-master-push
```

### Start Server
```
sh run_local.sh
```

### TODO
- NR Integration
- Monitoring instrumentation
- Recovery handler
- Remove logger

##  DB Migration
### One Time Complete Migration (Riddler -> Loki):
This follows the complete steps to migrate the data from Riddler DB(mysql) to Loki DB(Postgres).

1. Setup the configurations in ```config.yml```  file.  

2. Export the dump of ***godImages***, ***gods*** and ***appUserAttributes*** table from Riddler DB into  local as .csv files.  

	* ```select * from gods;``` ( [godImages table sample dump](https://drive.google.com/file/d/1CXEKBa4q9qhEGrwwfK44Aa1jkNZU3Ywp/view?usp=sharing) )  
	* ```select * from godImages;``` ( [god table sample dump](https://drive.google.com/file/d/1CXEKBa4q9qhEGrwwfK44Aa1jkNZU3Ywp/view?usp=sharing) )  
	* ```select userId as user_id, godOrder as god_order from appUserAttributes;``` ( [appUserAttributes sample dump](https://drive.google.com/file/d/1vdJvdNXWIcDTumYnKcyvWyIQZFp4aNLh/view?usp=sharing) )  

3. Copy the file path and assign the file paths to corresponding variables in ```scripts/migrations/migrate.py``` file.  
	like :-  
	***GOD_IMAGES_DUMP_FILEPATH*** : File path for god images table dump  
	***GOD_DUMP_FILEPATH*** : File path for god table dump  

4. Run the python script  
    ```python3 scripts/migrations/migrate.py```   
    > **Note:** dump file path will be printed on the screen, copy that.  

5. Run the following commands to create the table into the **loki** DB.  
	```
	./out/loki migrate --stream="down" --config-file config.yml
	make build
	./out/loki migrate --stream="up" --config-file config.yml
	```

6. Run the postgres copy command to populate the dump.Paste the file path into the following script, and run it.  
	* Populate ***gods*** table  
		filepath : File path copied from step 4
		```
		psql postgres -U minion -d loki -c "\copy gods (old_god_id,god_name,god_images,god_avatars,god_name_lang,alarm_song_id,position,is_active,default_god,user_id,is_user_uploaded,created_at,updated_at) from '<filepath_copied>' with null '' csv header;"
		```
	* Populate ***userGods*** table  
		filepath : File path of downloaded file for ***appUserAttibute*** table from step 2. 
		```
		psql postgres -U minion -d loki -c "\COPY gods (old_god_id,god_name,god_images,god_avatars,god_name_lang,alarm_song_id,position,is_active,default_god,user_id,is_user_uploaded,created_at,updated_at) from '<filepath_copied>' with null '' csv header;"
		```

### Quick Sample Migrate
This will directly populate the data into the local DB from sample dump files.  

1. Setup the configurations in ```config.yml```  file.  

2. Download the sample dump file  
	* [God Table Dump](https://drive.google.com/file/d/1oVi8ylci0EnBFjeBGHFYRpW9dF8rmBYa/view?usp=sharing)
	* [UserAppAttribute Table Dump](https://drive.google.com/file/d/1oVi8ylci0EnBFjeBGHFYRpW9dF8rmBYa/view?usp=sharing)

3.  Run the following commands to create the table into the **loki** DB.  
	```
	./out/loki migrate --stream="down" --config-file config.yml
	
	make build
	
	./out/loki migrate --stream="up" --config-file config.yml
	```

4. Run the postgres copy command to populate the dump.Paste the corresponding file path into the following script, and run it.  

	```
	psql postgres -U minion -d loki -c "\COPY user_attributes ("user_id", "god_order") FROM '<filepath>' WITH (FORCE_NULL(god_order), FORMAT csv, header);"
	psql postgres -U minion -d loki -c "\COPY gods (old_god_id,god_name,god_images,god_avatars,god_name_lang,alarm_song_id,position,is_active,default_god,user_id,is_user_uploaded,created_at,updated_at) from '<filepath_copied>' with null '' csv header;"
    ```
