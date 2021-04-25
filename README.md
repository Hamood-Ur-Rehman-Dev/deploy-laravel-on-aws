# Deploy Laravel On AWS
***Easy Steps To Deploy Laravel Project On AWS (BeanStalk, EC2, RDS)***

***Deployment | Domain Configuration | SSL Certificate (HTTPs)***

## Step 1: Deployment
Step 1.1: Create Elastic Beanstalk Application
Go to **Services** > **Elastic Beanstalk**
Create **New Application** [ add basic information and click **configure more options** ]
In Configure More Options, **EDIT Software** Section and set **Document Root** ***"/public"***
In Configure More Options, **EDIT Database** Section and set all required fields, Note down **UserName** & **Password** for *Database* we need this later.
Above Steps will create Application along with one **EC2 Instance** and **RDS Database**

Step 1.2: Create Key Pair & Bind With Elastic Beanstalk  Application
Go to **Services** > **EC2**
Select **Network & Security** > **Key Pair** from *Side Bar*
Create **New Key Pair**, this will automatically download key on your local machine.
Go to folder where ***YOUR_KEY.pem*** is located. **Open Terminal On Same Location** and write 
*``chmod 400 YOUR_KEY.pem``*
Now again Go Back To **Services** > **Elastic Beanstalk**
Go To **Select Your Environment** and from side bar **Click Configuration**
In **Elastic Beanstalk** > **Environments** > **YOUR_ENV_NAME** > **Configuration EDIT SECURITY** Section
Under Security Edit section and **Fill EC2 key pair** field with newly created Key **YOUR_KEY** and click apply
 
Above Steps will create **KEY** that allow you to access your **EC2 instance** using **ssh** and now this key is bind with **Elastic Beanstalk Environment**

### Step 1.3: Configure RDS (Database)
    >> - Go to **Services** > **RDS**
    >> - Click on **DB Instances** than click on **YOUR_RDS_NAME**
    >> - In **RDS** > **Databases** > **YOUR_RDS_NAME** go to **Connectivity & Security** Tab and under **Security** section click on **link under VPC security groups**
    >> - This will take you to **Security Groups** section of EC2 instance, Now Goto **Inbound Rules Tab** and click **Edit Inbound Rules** button.
    >> - Inside Inbound Edit page click **Add Rule** Button and add new Rule
    >> - New Rule will be **[ Type = All Traffic, Source = Anywhere ]** and Click Save Rules
    >> 
    >> Above Steps will allow you to **Access RDS** from your **EC2 Instance**
    >>
    >>
    >> ### Step 1.4: Connect Using SSH
    >> - Go to **Services** > **EC2** > **Instances (running)**
    >> - Click on **Checkbox Beside Running Instance** in list of instances in the table and click Connect button
    >> - Now **EC2** > **Instances** > **YOUR_EC2_INSTANCE** > **Connect To Instance** page is open Click **SSH Client Tab**
    >> - Copy The Link something that look something like this
    >>   - *``ssh -i "YOUR_KEY.pem" ec2-user@ec2-3-131-7-245.us-east-2.compute.amazonaws.com``*
    >> - Go to Folder where you save **YOUR_KEY.pem** file, open **Terminal** here and *Paste the command you have copied*.
    >> - After pressing **Enter** you are Connected to your EC2 instance
    >>   ![SSH TO EC2](/repo_screenshoots/step_1/ssh_ec2.png)
    >> - Your Project is located here, Go to this directory
    >>   - *``cd /var/app/current``*
    >>   ![Project Location](/repo_screenshoots/step_1/project_location.png)
    >> - Next we need to **Fix Routing Issue**, if now you open your project in browser you can only see **First Page** ***NO OTHER PAGE/ROUTE Works***, to solve this 
    >>   - *``sudo nano /etc/nginx/conf.d/elasticbeanstalk/php.conf``*
    >> - Add this line in **php.conf** file 
    >>   - *``location / {try_files $uri $uri/ /index.php?$args;}”``* 
    >>   - This will allow your routes to work properly
    >>   ![PHP Config File](/repo_screenshoots/step_1/php_conf.png)
    >> - Some time submitting form you get ***413 request entity too large*** error to resolve it 
    >>   - *``sudo nano /etc/nginx/nginx.conf``*
    >> - Add this line in **nginx.conf** file 
    >>   - *``client_max_body_size 20M;``* 
    >>   - inside **server { .. }** code block
    >>   ![Nginx Config File](/repo_screenshoots/step_1/nginx_conf.png)
___

> ## Step 2: Configure Domain 
    >> ## Step 2.1: Create Route53 
    >> - Goto **Services** > **EC2** 
    >>   - Make sure you have switched of **New EC2 Experience** switch. *(in new UI things may be in different locations)* 
    >>   - Right Click on **Your Instance** and click on **Networking** > **Change Security Groups** 
    >>   ![All EC2 Instances](/repo_screenshoots/step_2/check_security_group.png)
    >>   - Note Security **Security Group ID** OR **Security Group Name** we need it later *(copy Security Group ID)*
    >>   ![Attached Security Groups](/repo_screenshoots/step_2/note_security_group_details.png)
    >> - Now from *Side Navigation* goto **Network & Security** > **Security Groups** 
    >>   - Find Security Group that was attached with your instance. (past Security Group ID you copied earlier)
    >>   - Right click on Security Group and click on **Edit Inbound Rules**
    >>   ![All Security Groups](/repo_screenshoots/step_2/select_security_group.png)
    >>   - Click on **Add Rule** and Fill the details **[ Type = HTTP, Source = Anywhere]** and click **Save**
    >>   ![Add Inbound Rule](/repo_screenshoots/step_2/add_inbound_rule.png)
    >> - Now Goto **Services** > **Route53**
    >> - I am assuming that this is First Time You are on this Screen and Didn't have any Hosted Zone 
    >>   - Under **DNS Management** click on **Get Started Now**
    >>   ![Route53 Screen](/repo_screenshoots/step_2/route53_screen.png)
    >>   - Fill Required Filed by copying Domain Name. In my case GoDaddy
    >>   ![Route53 Screen](/repo_screenshoots/step_2/godaddy.png)
    >>   - Paste it under **Domain Name** 
    >>   ![Zone Creation](/repo_screenshoots/step_2/create_zone.png)
    >>   - After saving you will see this screen 
    >>   ![Zone Records](/repo_screenshoots/step_2/zone_key_values.png)
    >> - We have to create Type **A** record, so click on **New Record** button
    >>   - for **A** record we need **Public IPv4** of our **EC2 Instance**
    >>   ![Zone Records](/repo_screenshoots/step_2/IPv4_ec2.png)
    >>   - Leave **Record Name** field **empty**, **Record Type** select **A-Routes...** and in **Value** past **IPv4 of EC2** you copied
    >>   ![A Record](/repo_screenshoots/step_2/A_record.png)
    >> - Now next Create **CNAME** record, so again click on **New Record** button
    >>   - Leave **Record Name** field **www**, **Record Type** select **CNAME-Routes...** and in **Value** **your-domain.com** 
    >>   ![CNAME Record](/repo_screenshoots/step_2/CNAME_record.png)
    >> - By end of all this we have **4 records**
    >>   - Something like this
    >>   ![All HTTP Records](/repo_screenshoots/step_2/all_route53_records_http.png)
    >> - Next we have to **Update Name Severs**, click on record that have **Type NS**
    >>   - Copy name servers one-by-one **WITHOUT DOT in END**
    >>   - Goto your domain provider dashboard and in **DNS Management**
    >>   ![All HTTP Records](/repo_screenshoots/step_2/godaddy_dns_part_1.png)
    >>   - Find **Nameservers** section and click on **change**, set nameserver type **Custom** and Add Nameserves one-by-one you copied
    >>   ![All HTTP Records](/repo_screenshoots/step_2/godaddy_dns_part_2.png)
    >>   - This will take some time to by Domain Providers to update your domain servers



