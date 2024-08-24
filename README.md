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
      ![image](Image/12.jpg)
        - Full Name (Data, Mandatory)
        - Email Address (Data)
        - Phone (Data)
          Go to settings of library member doctype
    - Set **Naming** to `Autoincrement`.
      and fill name mem.####

      
    - Save the Doctype.

4 . add library member
![image](Image/11.jpg)

## Create Another Doctype: Library Membership

It will have the following fields:

- **Library Member** (Link, Mandatory) - Set options to Library Member
- **Full Name** (Data, Read Only) - Set fetch from to Library Member
- **From Date** (Date)
- **To Date** (Date)
- **Paid** (Check)

![image](Image/13.jpg)
  
Link the Library Member field to Library Member Doctype. Click on options and type Library Member.
### Fetching Value of Name from Library Member

The name will be automatically fetched from the Library Member doctype after selecting a Library Member.

It will have **Is Submittable** enabled and a Naming Series set as `LMS.#####`.

---
## Another Doctype: Library Transaction
![image](Image/14.jpg)
It will have the following fields:

- **Article** - Link to Article
- **Library Member** - Link to Library Member
- **Type** - Select with 2 options: Issue and Return
- **Date of Transaction** - Date
---

## Another Doctype: Library Settings
![image](Image/15.jpg)

It will have the following fields:

- **Loan Period** - Int
- **Maximum Number of Issued Articles** - Int

---
## Writing Code to Check Active Membership
Open Terminal

taranpreet@debian fm shell sdg24.com

- cd apps
- ls
- cd lib_man_sys
- ls
- cd lib_man_sys
- ls
- cd lib_man_sys
- ls
- cd doctype
- ls
- cd library_membership
- ls
- nano library_membership.py

## add code

```

import frappe
from frappe.model.document import Document
from frappe.model.docstatus import DocStatus


class LibraryMembership(Document):
    # check before submitting this document
    def before_submit(self):
        exists = frappe.db.exists(
            "Library Membership",
            {
                "library_member": self.library_member,
                "docstatus": DocStatus.submitted(),
                # check if the membership's end date is later than this membership's start date
                "to_date": (">", self.from_date),
            },
        )
        if exists:
            frappe.throw("There is an active membership for this member")

```





**lets make changes in library transaction doctype**
- cd ..
- ls
- cd library_transaction
- ls
- nano library_transaction.py

## Add the following code

```
import frappe
from frappe.model.document import Document
from frappe.model.docstatus import DocStatus


class LibraryTransaction(Document):
	def before_submit(self):
    	if self.type == "Issue":
        	self.validate_issue()
        	self.validate_maximum_limit()
        	# set the article status to be Issued
        	article = frappe.get_doc("Article", self.article)
        	article.status = "Issued"
        	article.save()

    	elif self.type == "Return":
        	self.validate_return()
        	# set the article status to be Available
        	article = frappe.get_doc("Article", self.article)
        	article.status = "Available"
        	article.save()

	def validate_issue(self):
    	self.validate_membership()
    	article = frappe.get_doc("Article", self.article)
    	# article cannot be issued if it is already issued
    	if article.status == "Issued":
        	frappe.throw("Article is already issued by another member")

	def validate_return(self):
    	article = frappe.get_doc("Article", self.article)
    	# article cannot be returned if it is not issued first
    	if article.status == "Available":
        	frappe.throw("Article cannot be returned without being issued first")

	def validate_maximum_limit(self):
    	max_articles = frappe.db.get_single_value("Library Settings", "max_articles")
    	count = frappe.db.count(
        	"Library Transaction",
        	{"library_member": self.library_member, "type": "Issue", "docstatus": DocStatus.submitted()},
    	)
    	if count >= max_articles:
        	frappe.throw("Maximum limit reached for issuing articles")

	def validate_membership(self):
    	# check if a valid membership exist for this library member
    	valid_membership = frappe.db.exists(
        	"Library Membership",
        	{
            	"library_member": self.library_member,
            	"docstatus": DocStatus.submitted(),
            	"from_date": ("<", self.date),
            	"to_date": (">", self.date),
        	},
    	)
    	if not valid_membership:
        	frappe.throw("The member does not have a valid membership")


```


      
## Enabling Web View for Article Doctype

1. Go to sdg24.com`
2. Login
3. `Ctrl+G`
4. Search Doctype List
5. Open Article
6. Go to the settings
7. Enable **Has Web View**
8. Allow Guest to view
9. Index Web Pages for Search
10. Set **Route** to `articles`
11. **Is Published** field to `published`
12. Click Save

Add two new rows in Fields:

- **Route** (Data)
- **Published** (Select)



## Customizing the Webpage

1. Open Terminal

```bash
cd article
cd templates
ls
nano article.html
```

Add the following code:

```html
{% extends "templates/web.html" %}

{% block page_content %}

<div class="py-20 row">
<div class="col-sm-2">
<img alt="{{ title }}" src="{{ image }}"/>
</div>
<div class="col">
<h1>{{ title }}</h1>
<p class="lead">By {{ author }}</p>
<div>
    {%- if status == 'Available' -%}
    <span class="badge badge-success">Available</span>
    {%- elif status == 'Issued' -%}
    <span class="badge badge-primary">Issued</span>
    {%- endif -%}
</div>
<div class="mt-4">
<div>Publisher: <strong>{{ publisher }}</strong></div>
<div>ISBN: <strong>{{ isbn }}</strong></div>
</div>
<p>{{ description }}</p>
</div>
</div>

{% endblock %}
```

Press `Ctrl+O`, then `Enter` to save and `Ctrl+X` to exit.

2. Edit `article_row.html`

```bash
nano article_row.html
```

Add the following code:

```html
<div class="py-8 row">
<div class="col-sm-1">
<img alt="{{ doc.name }}" src="{{ doc.image }}"/>
</div>
<div class="col">
<a class="font-size-lg" href="{{ doc.route }}">{{ doc.name }}</a>
<p class="text-muted">By {{ doc.author }}</p>
</div>
</div>
```

Press `Ctrl+O`, then `Enter` to save and `Ctrl+X` to exit.

Now, your webpage will have the updated design.

Thanks for reading!
```

