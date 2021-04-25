# Deploy Laravel On AWS
***Easy Steps To Deploy Laravel Project On AWS (BeanStalk, EC2, RDS)***

***Deployment | Domain Configuration | SSL Certificate (HTTPs)***

## Step 1: Deployment
* ***Step 1.1: Create Elastic Beanstalk Application***
  * Go to **Services** > **Elastic Beanstalk**
  * Create **New Application** [ add basic information and click **configure more options** ]
  * In Configure More Options, **EDIT Software** Section and set **Document Root** ***"/public"***
  * In Configure More Options, **EDIT Database** Section and set all required fields
  * Note down **UserName** & **Password** for *Database* you have to put those in **.env** file latter
    * DB_CONNECTION=mysql
    * DB_HOST=aqwtm3ea2ql7193.ccsrowesaut0.us-east-2.rds.amazonaws.com  (**Services > RDS > Your_RDS_instance > Connectivity & security Tab >** Under **Endpoint**)
    * DB_PORT=3306          
    * DB_DATABASE=ams_db (**Services > RDS > Your_RDS_instance > Configuration Tab >** Under **DB Name**)                                                    
    * DB_USERNAME=admin (**Services > RDS > Your_RDS_instance > Configuration Tab >** Under **Master username**)              
    * DB_PASSWORD=F$!(so)Hq
    * Above Steps will create Application along with one **EC2 Instance** and **RDS Database**

* ***Step 1.2: Create Key Pair & Bind With Elastic Beanstalk  Application***
  * Go to **Services** > **EC2**
  * Select **Network & Security** > **Key Pair** from *Side Bar*
  * Create **New Key Pair**, this will automatically download key on your local machine.
  * Go to folder where ***YOUR_KEY.pem*** is located. **Open Terminal On Same Location** and write 
    * *``chmod 400 YOUR_KEY.pem``*
  * Now again Go Back To **Services** > **Elastic Beanstalk**
  * Go To **Select Your Environment** and from side bar **Click Configuration**
  * In **Elastic Beanstalk** > **Environments** > **YOUR_ENV_NAME** > **Configuration EDIT SECURITY** Section
  * Under Security Edit section and **Fill EC2 key pair** field with newly created Key **YOUR_KEY** and click apply
  * Above Steps will create **KEY** that allow you to access your **EC2 instance** using **ssh** and now this key is bind with **Elastic Beanstalk Environment**

* ***Step 1.3: Configure RDS (Database)***
  * Go to **Services** > **RDS**
  * Click on **DB Instances** than click on **YOUR_RDS_NAME**
  * In **RDS** > **Databases** > **YOUR_RDS_NAME** go to **Connectivity & Security** Tab and under **Security** section click on **link under VPC security groups**
  * This will take you to **Security Groups** section of EC2 instance, Now Goto **Inbound Rules Tab** and click **Edit Inbound Rules** button.
  * Inside Inbound Edit page click **Add Rule** Button and add new Rule
  * New Rule will be **[ Type = All Traffic, Source = Anywhere ]** and Click Save Rules
  * Above Steps will allow you to **Access RDS** from your **EC2 Instance**

* ***Step 1.4: Connect Using SSH***
  * Go to **Services** > **EC2** > **Instances (running)**
  * Click on **Checkbox Beside Running Instance** in list of instances in the table and click Connect button
  * Now **EC2** > **Instances** > **YOUR_EC2_INSTANCE** > **Connect To Instance** page is open Click **SSH Client Tab**
  * Copy The Link something that look something like this
    * *``ssh -i "YOUR_KEY.pem" ec2-user@ec2-3-131-7-245.us-east-2.compute.amazonaws.com``*
  * Go to Folder where you save **YOUR_KEY.pem** file, open **Terminal** here and *Paste the command you have copied*.
  * After pressing **Enter** you are Connected to your EC2 instance
    * ![SSH TO EC2](/repo_screenshoots/step_1/ssh_ec2.png)
    * Your Project is located here, Go to this directory
      * *``cd /var/app/current``*
    * ![Project Location](/repo_screenshoots/step_1/project_location.png)
    * Next we need to **Fix Routing Issue**, if now you open your project in browser you can only see **First Page** ***NO OTHER PAGE/ROUTE Works***, to solve this 
      * *``sudo nano /etc/nginx/conf.d/elasticbeanstalk/php.conf``*
    * Add this line in **php.conf** file 
      * *``location / {try_files $uri $uri/ /index.php?$args;}”``* 
    * This will allow your routes to work properly
    * ![PHP Config File](/repo_screenshoots/step_1/php_conf.png)
    * Some time submitting form you get ***413 request entity too large*** error to resolve it 
      * *``sudo nano /etc/nginx/nginx.conf``*
    * Add this line in **nginx.conf** file 
      * *``client_max_body_size 20M;``* 
      * inside **server { .. }** code block
      * ![Nginx Config File](/repo_screenshoots/step_1/nginx_conf.png)
___
___

## Step 2: Configure Domain 
* ***Step 2.1: Create Route53***
  * Goto **Services** > **EC2** 
  * Make sure you have switched of **New EC2 Experience** switch. *(in new UI things may be in different locations)* 
    * Right Click on **Your Instance** and click on **Networking** > **Change Security Groups** 
        ![All EC2 Instances](/repo_screenshoots/step_2/check_security_group.png)
    * Note Security **Security Group ID** OR **Security Group Name** we need it later *(copy Security Group ID)*
        ![Attached Security Groups](/repo_screenshoots/step_2/note_security_group_details.png)
  * Now from *Side Navigation* goto **Network & Security** > **Security Groups** 
    * Find Security Group that was attached with your instance. (past Security Group ID you copied earlier)
    * Right click on Security Group and click on **Edit Inbound Rules**
       ![All Security Groups](/repo_screenshoots/step_2/select_security_group.png)
    * Click on **Add Rule** and Fill the details **[ Type = HTTP, Source = Anywhere]** and click **Save**
       ![Add Inbound Rule](/repo_screenshoots/step_2/add_inbound_rule.png)
  * Now Goto **Services** > **Route53**
    * I am assuming that this is First Time You are on this Screen and Didn't have any Hosted Zone 
    * Under **DNS Management** click on **Get Started Now**
        ![Route53 Screen](/repo_screenshoots/step_2/route53_screen.png)
    * Fill Required Filed by copying Domain Name. In my case GoDaddy
        ![Route53 Screen](/repo_screenshoots/step_2/godaddy.png)
    * Paste it under **Domain Name** 
        ![Zone Creation](/repo_screenshoots/step_2/create_zone.png)
    * After saving you will see this screen 
        ![Zone Records](/repo_screenshoots/step_2/zone_key_values.png)
  * We have to create Type **A** record, so click on **New Record** button
    * for **A** record we need **Public IPv4** of our **EC2 Instance** 
        ![Zone Records](/repo_screenshoots/step_2/IPv4_ec2.png)
    * Leave **Record Name** field **empty**, **Record Type** select **A-Routes...** and in **Value** past **IPv4 of EC2** you copied
        ![A Record](/repo_screenshoots/step_2/A_record.png)
  * Now next Create **CNAME** record, so again click on **New Record** button
    * Leave **Record Name** field **www**, **Record Type** select **CNAME-Routes...** and in **Value** **your-domain.com** 
        ![CNAME Record](/repo_screenshoots/step_2/CNAME_record.png)
  * By end of all this we have **4 records**
    * Something like this
        ![All HTTP Records](/repo_screenshoots/step_2/all_route53_records_http.png)
  * Next we have to **Update Name Severs**, click on record that have **Type NS**
    * Copy name servers one-by-one **WITHOUT DOT in END**
    * Goto your domain provider dashboard and in **DNS Management**
        ![GoDaddy DNS](/repo_screenshoots/step_2/godaddy_dns_part_1.png)
  * Find **Nameservers** section and click on **change**, set nameserver type **Custom** and Add Nameserves one-by-one you copied
        ![GoDaddy DNS](/repo_screenshoots/step_2/godaddy_dns_part_2.png)
  * This will take some time to by Domain Providers to update your domain servers
___
___

## Step 3: SSL Certificate (HTTPs)
* ***Step 2.1: Create Certificate***
  * Goto **Services** > **Certificate Manager**.
  * Under **Provision Certificate** click on **Getting Started**.
    ![Certificate Manager](/repo_screenshoots/step_3/certificate_manager.png)
    * For Certificate Type select **Request a public certificat**  (FREE SSL). 
        ![Certificate Type](/repo_screenshoots/step_3/certificate_type.png)
    * Add Domain for your certificate.
        ![Certificate Add Domain](/repo_screenshoots/step_3/certificate_add_domain.png)
    * Select Certification Validation Method.
        ![Certificate Validation Method](/repo_screenshoots/step_3/certificate_validation_method.png)
    * **Tags** are optional so leave as **default** and click Next.
    * Review your data.
        ![Certificate Review](/repo_screenshoots/step_3/certificate_review.png)
    * Next Validation Page, Wait 5min until your Validation Status change to **ISSUED** see current status is **Pending validation**.
        ![Certificate Validation](/repo_screenshoots/step_3/certificate_validation_page.png)
        * Copy **CNAME** **Name** *(WITHOUT DOMAIN_NAME at the end)* and **Value** that look like **f2793....** **_7ca09....** respectively we need to create another **CNAME record** in **Route53** that require this.
  
* ***Step 2.2: Configure Route53 for HTTPs***
  * Goto **Services** > **Route53**.
  * Create another **CNAME Record**, paste CNAME you copied without **YOUR_DOMAIN** and select **Record Type** ***CNAME-Routes traffic to another domain name and to some AWS Resources***  and toggle ***Alias switch*** and ***Set Alias to another record in this hosted zone*** next choose region and in last ***Choose Record*** ***YOUR_DOMAIN_NAME*** and click on Create Record.
    ![HTTPS CNAME Record](/repo_screenshoots/step_3/CNAME_HTTPs.png)

* ***Step 2.3: Create Load Balancer***
  * Goto **Services** > **EC2** > In Side Nav **Load Balancers**.
  * Click on **Create Load Balancer** and on next screen **Application Load Balancer** section click on **Create**.
    ![Load Balancer](/repo_screenshoots/step_3/load_balancer_page.png)
  * **Step 2.3.1:** will be some basic configuration like **Name**, **Listener** & **Availability Zones**.
    ![Load Balancer Step 1](/repo_screenshoots/step_3/load_balancer_step_1.png)
    ![Load Balancer Step 1_1](/repo_screenshoots/step_3/load_balancer_step_1_1.png)
  * **Step 2.3.2 Configure Security Settings:** From drop-down of **Certificate name** select **Your Certificate** you created in last step.
  * **Step 2.3.3 Configure Security Groups:** Select **application security** groups.
  * **Step 2.3.4 Configure Routing:** Set **Name** as you like and leave other fields as they are and click on Next Register Targets.
  * **Step 2.3.5 Register Targets:** Under **Instances** click on **Your Running Instance** and click on **Add To Registered**.

* ***Step 2.4: Configure Route53 for HTTPs Again***
  * Goto **Services** > **Route53**.
  * Create another **A Record**, Leave **Name** **Empty** and set **Record Type** to **A-Route Traffic to an IPv4 address and some AWS Resources**. Check **Alias toggle** button. And select **Alias to Application and Classic Load Balancer** and next set **Your Region** that will show you available Load Balancers for that region select the **Load balancer you created** and Click **Create Record**

* ***Step 2.5: Configure Route53 for HTTPs Again***
  * Now go to **EC2** > **Load Balancer** and select your load balancer instance and goto **Listener Tab**
  * Now **Edit HTTP:80** ,**Delete Record** under **Default Actions** and click on **Add Action**, Select **Redirect To** and select **HTTPS** and **port 443** and click on **TICK** button to add record. Now click on **Update**. Thats all.

* **Note:** After shifting to HTTPs you will notice that **CSS**, **Images** and all other **Resources crashes**. because all resources are still getting **http://** link to fix this 
  * Open your laravel application code and goto **AppServiceProvider** and in **boot()** method add this line
    * *``\URL::forceScheme(‘https’);``*