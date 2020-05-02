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
35. Your application is now running: http://ec2-15-206-178-249.ap-south-1.compute.amazonaws.com:8000

