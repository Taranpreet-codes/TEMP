### Tutorial:Setting Up a Library Management System

**Prepared by**:Taranpreet Kaur & Kusum  
  

#### Setting Up Domain and Basic Configurations

1. **Create and Start Site**:
    ```bash
    fm create sdg24.com
    fm start sdg24.com
    ```

2. **Edit Hosts File**:
    ```bash
    su
Taranpreet-codes/TEMP
￼
 main
￼
Go to file

    Password:
    root@debian:~# nano /etc/hosts
    ```
    Add the following lines:
    ```plaintext
    127.0.0.1    localhost
    127.0.0.1    sdg24.com
    ```
    Exit root:
    ```bash
    root@debian:~# exit
    ```

3. **Enter Frappe Shell**:
    ```bash
    cd frappe
    taranpreet@debian:~/frappe$ fm shell sdg24.com
    ```

#### Creating and Installing the App

1. **Create New App**:
    ```bash
    bench new-app lib_man_sys
    ```

2. **Install App**:
    ```bash
    bench install-app lib_man_sys
    ```

3. **Access the App**:
    - Open a browser and go to `sdg24.com`.
   
     ![image](Image/frappe_homepage.jpg)
 - Login with `username = administrator` and `password = admin`.

 ![image](Image/frappe_homepage_1.jpg)   
  - Click on the profile icon (top right), then select "Switch to Desk".
![image](Image/2.jpg)
#### Creating Doctypes

1. **Create Doctype: Article**
![image](Image/3.jpg) 
    - **Module**: Lib Man Sys
    - **Fields**:
        - Article Name (Data, Mandatory)
        - Image (Attach Image)
        - Author (Data)
        - Description (Text Editor)
          ![image](Image/4.jpg)
        - ISBN (Data)
        - Status (Select: Issued, Available)
          ![image](Image/5.jpg)
        - Publisher (Data)
    - Save the Doctype.
 go to article list

      ![image](Image/6.jpg)
      Add article
      ![image](Image/7.jpg)
      Go to settings of article doctype
      add roles
      ![image](Image/8.jpg)
      
      

3. **Create Doctype: Library Member**
   ![image](Image/9.jpg)
    - **Fields**:
      ![image](Image/10.jpg)
        - Full Name (Data, Mandatory)
        - Email Address (Data)
        - Phone (Data)
          Go to settings of library member doctype
    - Set **Naming** to `Autoincrement`.
      and fill name mem.####

      
    - Save the Doctype.

4 . add library member
![image](Image/11.jpg)
