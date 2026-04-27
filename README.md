# # COMP-3297-2026-group-D BetaTrax Sprint 3

## Project Overview
BetaTrax is a defect tracking system developed using **Django 6.0.2** and **Django Rest Framework (DRF)**.

For submission-ready API documentation, see [docs/API_DOCUMENTATION.md](docs/API_DOCUMENTATION.md).


## Sprint 3 Enhancements over Sprint 1 & 2
- **Multi-tenant support** – Added tenant isolation to enable BetaTrax to be rolled out as a SaaS product for other development companies, as described in the Project Description.

- **Developer effectiveness metric** – Implemented a simple metric to calculate and report the effectiveness of individual developers in fixing defects.

- **Basic automated tests** – Added a suite of basic automated tests for the Django web service, covering models, forms, and key API endpoints.

- **Tests + coverage for developer effectiveness classification** – Wrote dedicated tests for the code that classifies developer effectiveness, and used a coverage tool to demonstrate that those tests meet specified adequacy criteria .

- **User ID visibility & API usage** – Each user's User ID is now visible in the admin system. When requesting the effectiveness metric for a particular developer, the API expects the developer's User ID to be provided as a parameter.

- **Fixed email notification issue** – Resolved the problem from Sprint 2 where notification emails were not actually being sent.

- **Fixed duplicate notification issue** – Resolved the problem where tester_email merging was incorrectly treated the same as cascade notifications; they are now handled separately.


## How to Run
1. Activate virtual environment: run `python -m venv venv` (or `python3 -m venv venv` for MacOS and Linux) and then run `.\venv\Scripts\activate` (or `. venv/bin/activate` for MacOS and Linux) and then run `pip install Django==6.0.2 djangorestframework django-filter`. Lastly, run `pip install -r requirements.txt` to ensure all required files are installed.
2. Run the server: `python manage.py runserver`
3. Create new superuser(admin): `python manage.py createsuperuser`, and provide username and password for the admin of the app.
4. Root URL: http://127.0.0.1:8000/ now redirects to the API landing page at http://127.0.0.1:8000/api/  
5. Admin panel: http://127.0.0.1:8000/admin/  
6. API endpoint: http://127.0.0.1:8000/api/defects/ -->this one for developers and owners to review the reports
                 http://127.0.0.1:8000/api/products/ -->this one for owners to add new products
7. View defect reports: Open http://127.0.0.1:8000/api/defects/<id>/ (e.g., http://127.0.0.1:8000/api/defects/1/) in your browser after logging in.
8. View products: Open http://127.0.0.1:8000/api/products/<id>/ (e.g., http://127.0.0.1:8000/api/products/1/) in your browser after logging in.
9. View developer metrics: Open http://127.0.0.1:8000/api/defects/metrics/<user_id>/ (e.g., http://127.0.0.1:8000/api/defects/metrics/12/) in your browser after logging in (The user_id here must be a the id of a developer).

## Automation Test Instructions (Sprint 3)
**Preconditions:**
1) Python/Python3 intsalled
2) Coverage installed (run `pip install coverage` to install coverage)

**Run Instructions(API Test):**
1) Do [How to Run](#How-to-Run) Step 1
2) Run `python manage.py test defects.test_api`

**Run Instructions(Developer Metrics Classification Logic Test):**
1) Do [How to Run](#How-to-Run) Step 1
1) Run "python manage.py test defects.test_dev_metrics" for normal testing

**Generate Coverage Report (Developer Metrics Classification Logic Test):**
1) Run `coverage run --source='defects' manage.py test defects.test_dev_metrics`
2) Run `coverage report --include="views.py"` to generate report
3) Run `coverage report -m --include="view.py"` to see how many line the testing misses
4) Run `coverage html --include="views.py"` to generate the report in html
5) Run `open htmlcov/index.html` / `xdg-open htmlcov/index.html` / `start htmlcov/index.html` to view report in `MacOS` / `Linux` / `Windows`


## Limitations (Sprint 3)

1) **Incompatibility of test_api.py** -- The current version of automation test for all API endpoints only work for the Sprint 2 application (using SQLite). The program is unable to interact with the new SQL (PostgreSQL). Due to time constraints, we were unable to update the automation code for API test to be compatable with PostgreSQL and all API requests from test_api.py to the SQL returns a "404 not found" error.

--
## How to Run PostgreSQL Multi-Tenants (Local Development)

1. **Set up the database**
   - Make sure PostgreSQL is running. Then create the database and user (example using psql):

     ```sql
     CREATE USER betatrax_user WITH PASSWORD 'your_password';
     CREATE DATABASE betatrax_db OWNER betatrax_user;
     ALTER USER betatrax_user CREATEDB;
     ```

     *Note: Update line 67 in settings.py with the PASSWORD set to 'your_password'*

2. **Clone / unzip the project and create a virtual environment**

   ```bash
   python -m venv venv
   # Windows
   .\venv\Scripts\activate
   # macOS/Linux
   source venv/bin/activate
   ```

3. **Install dependencies**

   ```bash
   pip install -r requirements.txt
   ```

   *(requirements.txt includes Django, DRF, django-tenants, psycopg2-binary, coverage, etc.)*

4. **Configure database (if needed)**
   - Open `BetaTrax/settings.py` and verify the DATABASES section matches your PostgreSQL credentials.
   - The engine must be `django_tenants.postgresql_backend`.

5. **Apply migrations**

   ```bash
   python manage.py makemigrations tenants defects
   python manage.py migrate_schemas
   ```

6. **Create a superuser (shared across all tenants)**

   ```bash
   python manage.py createsuperuser
   ```

7. **Create your first tenant and domain**

   ```bash
   python manage.py shell
   ```

   Inside the shell:

   ```python
   from tenants.models import Client, Domain

   tenant = Client(schema_name='dev', name='Dev Company')
   tenant.save()

   domain = Domain()
   domain.domain = 'dev.localhost'
   domain.tenant = tenant
   domain.is_primary = True
   domain.save()
   exit()
   ```

8. **Update your local hosts file**
   - Add the following line to the last row of your hosts file:
     - Windows: `C:\Windows\System32\drivers\etc\hosts`
     - macOS/Linux: `/etc/hosts`

     ```
     127.0.0.1  dev.localhost
     ```

9. **Run the server**

   ```bash
   python manage.py runserver
   ```

10. **Access the application**
    - Admin panel: http://dev.localhost:8000/admin/ (use the superuser you created)
    - API endpoints:
      - http://dev.localhost:8000/api/defects/
      - http://dev.localhost:8000/api/products/

To add more tenants (e.g., test1.localhost), repeat steps 7–8 with a different schema_name and domain.

