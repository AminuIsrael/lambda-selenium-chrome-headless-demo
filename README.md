# Selenium with Headless Chrome on AWS Lambda

## Deployment Steps
1. Go to AWS Lambda, choose your preferred region and create a new function.

2. Choose `Author from scratch`. Give it a function name, and choose Python 3.6 as the runtime. Under `Role`, choose `Create new role from template`. Roles define the permissions your function has to access other AWS services, like S3. For now give this role a name like `scraper`. Under `Policy templates` choose `Basic Edge Lambda permissions`, which gives your function the ability to run and create logs. Hit `Create function`.

3. Now you’re in the main Lambda dashboard. Here is where you set up the triggers, environment variables, and access the logs. Since we want this to run on a schedule, we need to set up the trigger. On the `add trigger` menu on the left, choose `CloudWatch Events`.

    Click on `Configuration required` to set up the time the script will run. A form will appear below it.

    Under `Rule` choose `Create a new rule`. Give it a rule name. It can be anything, like `dailyscrape`. Give it a description so you know what it’s doing if you forget.

    Now you have to write the weird cron expression that tells it when to run. I want to scraper to run at 11 PM EST every night. In Lambda, you need to enter the time in GST, so that’s 3 AM. My expression therefore needs to be: `cron(0 3 * * ? *)`

    Click on `Add` and that’s it. Your function is set to run at a scheduled time.

4. We need to set up our environment variables. Since we’ll be uploading a Chrome browser, we need to tell Lambda where to find it. So punch in these keys and values.

    `PATH = /var/task/bin`<br>
    `PYTHONPATH = /var/task/src:/var/task/lib`

5. Under `Basic settings` choose how much memory will be allocated to your scraper. My scraper rarely needs more than 1000 MB, but I give it a little more to be safe. If the memory used goes above this limit, Lambda will terminate the function.

    Same with execution time. Lambda gives you a maximum of fifteen minutes to run a function.

6. Finally, you need to give your file and function a name that Lambda will know how to run. By default, Lambda will look for a file called `lambda_function.py` and run a function called `lambda_handler`. You can call it whatever you want, just make sure you change this in the `Lambda dashboard` (under `Handler`) and in your Makefile, under the `docker-run` command.

    So if your file is called `crime_scraper.py` and your main function is called `main()`, you need to change these values to `crime_scraper.main`

7. Create the zip package by running below command

    `make build-lambda-package`

    The resulting file will be too big to upload directly to Lambda, since it has a limit of `50MB`. A Selenium scraper is big, because you need to include a Chrome browser with it. So you’ll need to add it to an S3 bucket and tell Lambda where it is. So choose `Upload file form Amazon S3` in Function area and upload the zip file from S3 to lambda.

<br><br>
## Reference
https://robertorocha.info/setting-up-a-selenium-web-scraper-on-aws-lambda-with-python/