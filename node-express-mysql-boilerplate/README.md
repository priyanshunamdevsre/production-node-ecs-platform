# Node Web Application boilerplate

A boilerplate for **Node.js** web applications. This boilerplate gives the basic stucture of application start with while bundling enough useful features so as to remove all those redundant tasks that can derail a project before it even really gets started. This boilerplate users Express with sequelize as ORM and MySQL as database.

### Prerequisites

1. **Node.js** (v14 or higher recommended)  
2. **npm** (comes bundled with Node.js)  
3. **MySQL** (v5.7 or higher)

### Quick start

1. Clone the repository with `git clone https://github.com/priyanshunamdevsre/production-node-ecs-platform.git <your_project_folder_name>`
2. Change directory to your project folder `cd <your_project_folder_name>`
3. Install the dependencies with `npm install`
4. Create database in MySQL.
   - mysql -u root -p
   - CREATE DATABASE node_express_app;
   - EXIT;

5. Update the your database name and credentials in the `.env` file.
6. Run the application with `npm start` (MySQL service should be up and running).
7. Access `http://localhost:3000` and you're ready to go!
