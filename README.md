# AwsDemoWebApp

Steps for deploying Django web app to AWS EC2:

1. Create a EC2 (Ubuntu) instance in AWS and download the keypair.
2. Run Django Project on local machine Link to sample application repo: git clone https://github.com/akashishu777/AwsDemoWebApp.git
3. Update the setting.py 

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
    }
}

becuase we are not using database

4. install the reqirement.txt by running the cmd pip install -r requirements.txt
5. Spin up the server by running python manage.py runserver
6. check the deployment status by runnign cmd python manage.py check --deploy
7. Go to setting.py and disable the debug mode (DEBUG = False)
8. set ALLOWED_HOSTS = ['*'] in setting.py
9. Put your keypair to root dir of project, and open git bash
10. make the key read only ny runnign cmd (chmode 400 testkeyapir.pem)
11. copy the ssh cmd and run (ssh -i "test-keypair-django.pem" ubuntu@ec2-15-206-178-249.ap-south-1.compute.amazonaws.com)
now you are conencted to server
12. udpate the apt (sudo apt-get update)
13. Check the python version (python3 --version)
14. Install venv (python3 -m venv env)
now you will get error Failing command: ['/home/ubuntu/env/bin/python3', '-Im', 'ensurepip', '--upgrade', '--default-pip']

15. now install (sudo apt-get install python3-venv)
16. now create a virtaul enviroment (python3 -m venv env)
17. now use this as our enviroment (source env/bin/activate)
18. now install django using cmd (pip3 install django)
19. Check git version and clone your project (git clone https://github.com/akashishu777/AwsDemoWebApp.git)
20. run cmd (ls) you will see your folder, now change the dir to AwsDemoWebApp cmd (cd AwsDemoWebApp)
21. now instal gunicorn cmd (pip3 install gunicorn)
22. now install nginx cmd (sudo apt-get install -y nginx)
23. now start nginx cmd (sudo nginx) but it will not work when you visit http://ec2-15-206-178-249.ap-south-1.compute.amazonaws.com/
this is becuas we need to assign our server to security group, for allowing trafic.

24. check security group, right click on Djngo test in aws -> Networking -> chnage security group you will see launch-wizard-1
25. now configure the security group, Go to security group under network & Security side bar
26. Choose your group name launch-wizard-1 and edit Inbound rule.
27. Click on Add rule -> change custome TCP to HTTP -> Change the source to anywhere
28. now when you open your Public DHS (http://ec2-15-206-178-249.ap-south-1.compute.amazonaws.com/) you will see (Welcome to nginx!)
29. now we have to connect gunicorn to nginx
30. Go to terminal, cd TestProject then (ls) you will see wsgi.py
31. Now we need to tell gunicorn use wsgi.py file inorder to serve django application rather then nginx.
32. Run the cmd (gunicorn --bind 0.0.0.0:8000 TestProject.wsgi:application)
33. now visit http://ec2-15-206-178-249.ap-south-1.compute.amazonaws.com:8000/  but you will not see anything becuase inbound trafic are only allowed for 80 port lets add new inbound with 8000 port in aws
34. Go to EC2 > Securitygroup > Edit Inbound rules add (Cutom TCP -> port range: 8000 -> source anywhere) save it.

Your application is now running: http://ec2-15-206-178-249.ap-south-1.compute.amazonaws.com:8000

35. Now we will install supervisor for running the app in background cmd (sudo apt-get install -y supervisor)
36. now we need to create a configuration so that supersior read from the config and appropreatly restart/start the application and where to log the error.
37. Now switch to -> cmd (cd /etc/supervisor/conf.d/) you are in now (env) ubuntu@ip-172-31-14-42:/etc/supervisor/conf.d$
38. Create configuration file cmd (sudo touch gunicorn.conf) run ls for seeing created file.
39. Now edit the config file cmd (sudo nano gunicorn.conf)

[program:gunicorn]
directory=/home/ubuntu/AwsDemoWebApp
command=/home/ubuntu/env/bin/gunicorn --workers 3 --bind unix:/home/ubuntu/AwsDemoWebApp/app.sock TestProject.wsgi:application
autostart=true
autorestart=true
stderr_logfile=/var/log/gunicorn/gunicorn.err.log
stdout_logfile=/var/log/gunicorn/gunicorn.out.log

[group:guni]
programs:gunicorn

40. save and exit cmd (ctrl + O   press enter then  ctrl + X)
41. create dir which we mentioned above in the code cmd (sudo mkdir /var/log/gunicorn)
42. now lets tellsupervisor to read this file cmd (sudo supervisorctl reread)
43. now lets tell the superviror to start the guni cmd (sudo supervisorctl update) msg (guni: added process group)
44. lets check the status now cmd (sudo supervisorctl status) now in the background gunicorn is running.
45. Next steps is to congifure nginx to read from this scoket file 
46. cmd (cd) go back to hone directory 
47. cd /etc/nginx/sites-available/   and see the files by cmd (ls)
48. cat default (just sample config file)
49. now lets create a new file for our application cmd (sudo touch django.conf)
50. sudo nano django.conf

server {
	listen 80;
	server_name ec2-15-206-178-249.ap-south-1.compute.amazonaws.com;

	location / {
		include proxy_params;
		proxy_pass http://unix:/home/ubuntu/AwsDemoWebApp/app.sock;
	}
}

exit cmd (sudo nano django.conf)
now paste this code in django.conf and save it with cmd (CTRL+ O & CTRL + X)

51. now lets test this conf cmd (sudo nginx -t) but this is not reading our conf file.
52. lets enable this site cmd (sudo ln django.conf /etc/nginx/sites-enabled/)
53. sudo service nginx restart

Tutorial: https://www.youtube.com/watch?v=u0oEIqQV_-E
