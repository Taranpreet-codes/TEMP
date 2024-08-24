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
    - Login with `username = administrator` and `password = admin`.
    - Click on the profile icon (top right), then select "Switch to Desk".
    
     ![image](frappe homepage.jpg)

#### Creating Doctypes

1. **Create Doctype: Article**
    - **Module**: Lib Man Sys
    - **Fields**:
        - Article Name (Data, Mandatory)
        - Image (Attach Image)
        - Author (Data)
        - Description (Text Editor)
        - ISBN (Data)
        - Status (Select: Issued, Available)
        - Publisher (Data)
    - Save the Doctype.

2. **Create Doctype: Library Member**
    - **Fields**:
        - Full Name (Data, Mandatory)
        - Email Address (Data)
        - Phone (Data)
    - Set **Naming** to `Autoincrement`.
    - Save the Doctype.

3. **Create Doctype: Library Membership**
    - **Fields**:
        - Library Member (Link to Library Member, Mandatory)
        - Full Name (Data, Read Only)
        - From Date (Date)
        - To Date (Date)
        - Paid (Check)
    - **Link** the "Library Member" field to the "Library Member" Doctype.
    - Set **Naming** to `LMS.#####`.
    - Enable **Is Submittable**.
    - Save the Doctype.

4. **Create Doctype: Library Transaction**
    - **Fields**:
        - Article (Link to Article)
        - Library Member (Link to Library Member)
        - Type (Select: Issue, Return)
        - Date of Transaction (Date)
    - Save the Doctype.

5. **Create Doctype: Library Settings**
    - **Fields**:
        - Loan Period (Int)
        - Maximum Number of Issued Articles (Int)
    - Save the Doctype.

#### Adding Business Logic

1. **Check for Active Membership**:
    - Open `library_membership.py`:
      ```bash
      cd ~/frappe-bench/apps/lib_man_sys/lib_man_sys/lib_man_sys/doctype/library_membership
      nano library_membership.py
      ```
    - Add the following code:
      ```python
      import frappe
      from frappe.model.document import Document
      from frappe.model.docstatus import DocStatus

      class LibraryMembership(Document):
          def before_submit(self):
              exists = frappe.db.exists(
                  "Library Membership",
                  {
                      "library_member": self.library_member,
                      "docstatus": DocStatus.submitted(),
                      "to_date": (">", self.from_date),
                  },
              )
              if exists:
                  frappe.throw("There is an active membership for this member")
      ```
    - Save and exit.

2. **Transaction Validations**:
    - Open `library_transaction.py`:
      ```bash
      cd ../library_transaction
      nano library_transaction.py
      ```
    - Add the following code:
      ```python
      import frappe
      from frappe.model.document import Document
      from frappe.model.docstatus import DocStatus

      class LibraryTransaction(Document):
          def before_submit(self):
              if self.type == "Issue":
                  self.validate_issue()
                  self.validate_maximum_limit()
                  article = frappe.get_doc("Article", self.article)
                  article.status = "Issued"
                  article.save()

              elif self.type == "Return":
                  self.validate_return()
                  article = frappe.get_doc("Article", self.article)
                  article.status = "Available"
                  article.save()

          def validate_issue(self):
              self.validate_membership()
              article = frappe.get_doc("Article", self.article)
              if article.status == "Issued":
                  frappe.throw("Article is already issued by another member")

          def validate_return(self):
              article = frappe.get_doc("Article", self.article)
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
    - Save and exit.

3. **Auto-Compute Loan Period**:
    - Open `library_membership.py`:
      ```bash
      cd ../library_membership
      nano library_membership.py
      ```
    - Add the following code at the end:
      ```python
      loan_period = frappe.db.get_single_value("Library Settings", "loan_period")
      self.to_date = frappe.utils.add_days(self.from_date, loan_period or 30)
      ```
    - Save and exit.

#### Web View Customization

1. **Enable Web View**:
    - Go to `anmol.com`, login, and open the "Article" Doctype.
    - In settings, enable "Has Web View," allow guest viewing, and index web pages for search.
    - Set `Route` as `articles`, and `Is Published` field to `published`.

2. **Customize Web Page**:
    - Open the terminal and navigate to the `article` templates:
      ```bash
      cd ~/frappe-bench/apps/lib_man_sys/lib_man_sys/lib_man_sys/doctype/article/templates
      nano article.html
      ```
    - Add the following code:
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
    - Save and exit.

3. **Customize Article Row**:
    - Open `articles_row.html`:
      ```bash
      nano articles_row.html
      ```
    - Add the following code:
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
    - Save and exit.

4. **View Published Articles**:
    - Visit `anmol.com/articles` to see the published articles with their customized view.

---

This guide walks you through setting up a Library Management System using Frappe. Follow each step to create and customize the app, manage your library's articles and memberships, and enable a web view for public access.
