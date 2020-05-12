# todowebapp
## Table of Contents
[Introduction](https://github.com/faimoh/todowebapp#todowebapp)

[Technologies Used](https://github.com/faimoh/todowebapp#technologies-used)

[Requirements Analysis](https://github.com/faimoh/todowebapp#requirements-analysis)

[Data Model](https://github.com/faimoh/todowebapp#data-model)

[Application Architecture](https://github.com/faimoh/todowebapp#application-architecture)


## Introduction
A Java based web application for ToDo activities. Through this web application, one can create, read and update their todos through a modern web browser. The application also implements AAA, which means every user has their own account and their todo list etc. are private to them.

This document is not for the newbies. You must possess good knowledge of below technologies to understand this document and the related application:
1. Java
2. OOA&D
3. SQL, RDBMS
4. HTML
5. HTTP
6. Servlet, JSP
7. Tomcat


## Technologies Used
Java  
Servlet, JSP  
Apache Tomcat  
MySQL  
HTML
Apache NetBeans IDE
Firefox

This is entirely a back-end project. So, front-end technologies like CSS, JavaScript are not used. The aim of the project is to effectively learn and showcase how different pieces of the Java Servlet API work together.

We shall develop the web application starting with requirements analysis. Then we shall move on to database design. Data is central to any web application. Almost all use cases deal with data. Once the web application's data model is ready, we shall then move on to design the architecture of the application. In this phase, we shall see how our application behaves to different HTTP actions. Because, all actions performed by the application's users are through HTTP. We shall think of all possible user actions and define them clearly. Next, we shall move on to designing the interfaces and classes.

## Requirements Analysis
For our application, we begin with defining what a todo is for us. A todo is a task that must be accomplished. We create list of such tasks to help us live a prdocutive life. We keep tracking the list as we get the tasks done one after the other. A todo item for us has below properties:
1. an unique identification number such as a positive number
2. details of the task that we want to accomplish
3. task created timestamp
4. task status

Initially, a task will have the status 'todo'. When we start working on it, we change it to 'in progress'. Once the task gets accomplished, we mark its status as 'done'.

Also, we don't want to specify a deadline for a task that we create. We just want a simple todo list.

We want our application to also support multiple users. And every user shall have their own private list. Thus, users cannot see others' todo list. A user shall be identified by their username, which is their valid email address for us. Users are given accounts on our application. Thus, an account has below properties:
1. an unique account identification number such as a positive integer
2. a distince username, which is an email address. Duplicate email addresses are not allowed.
3. user's first name. Duplicates are allowed.
4. user's last name. Duplicates are allowed.
5. password. Duplicates are allowed.
6. account status - enabled/disabled. Only enabled user accounts will be able to login the application.

We want an 'administrator' account to only manage accounts. The administrator account shall use the username 'admin'. The admin user can:
1. create a user account
2. help reset password in case someone forgets
3. can enable or disable a user account
4. not disable 'admin' account
5. change user details
6. not have access to any user's todo lists
7. not maintain a personal todo list

The last two items are worth observing. Usually, it is believed that a user with 'admin' rights has access to everybody's information. We don't want such. Also, we have already defined that 'admin' account for us is only to manage accounts. It is not for managing todo lists of users. 'admin' user account is not often used. It is meant only for special purposes. For our application, we expect one user account to also handle 'admin' account. So, it will be the same person who logs in using 'admin' credentials only when required. Because it is an existing user account using 'admin' account only to manage all accounts, we don't want a separate task list for admin account. That doesn't serve any purpose.

We want the todo lists to persist always. Which means, once a todo item is created successfully by a user, it can never be deleted. Similarly, we also don't want to delete a user account. In conclusion, we don't want to support 'delete' operations in our application. Thus, we only support CRU out of CRUD.

Because, we want our application to maintain private todo lists, we want the application to provide login and logout mechanisms. This is called 'authentication'. Every user, including 'admin', should first authenticate themselves. Upon successful authentication, a user will be redirected to their workspace. Because we are discussing two types of users (one admin and the other normal), we shall have two types of workspaces on our application. An admin user shall be only working with user account management workspace. A normal user shall be only working with todo list management workspace. Both are exclusive. A normal user cannot see admin's workspace. And admin user cannot see normal user's workspace. This is called 'authorization'.

In addition to the above requirements, we want our application to store the details of users' login and logout timestamps. Through this we are tracking users' activity on our application. This is not exactly 'accounting' from AAA, but for our application this serves the purpose of calling it as 'accounting'.

## Data Model
Based on the requirements that we have collected so far, we understand that we have to store data for below entities of the application:
1. account
2. task

Example data for few accounts:
|Account ID|Username|First Name|Last Name|Password|Status|
|---|---|---|---|---|---|
|1|admin|Administrator|User|password|enabled|
|2|john@example.com|John|Johnsson|oneword|disabled|
|3|eric@example.com|Eric|Ericsson|twoword|enabled|
|4|ana@example.com|Ana|Mary|threeword|enabled|

We see that account statuses are repeated throughout the table. So, as part of database normalization, it's better to put the repeated data in a separate table. There's some good reason behind. Let's say, we have 100 users. And we want to replace the words enabled and disabled with 1 and 2 respectively. We have to modify the status column of all rows of the table. Imagine how cumbersome it will be to do such modification for a table with thousands of rows! Database normalization at rescue, thankfully!

After normalization, we shall have two tables - account_statuses and accounts:

**ACCOUNT_STATUSES**
|ID|Status|
|---|---|
|1|enabled|
|2|disabled|

**ACCOUNTS**
|Account ID|Username|First Name|Last Name|Password|Status ID|
|---|---|---|---|---|---|
|1|admin|Administrator|User|password|1|
|2|john@example.com|John|Johnsson|oneword|2|
|3|eric@example.com|Eric|Ericsson|twoword|1|
|4|ana@example.com|Ana|Mary|threeword|2|

Similarly, we shall have two tables for tasks - task_statuses and tasks: 

**TASK_STATUSES**
|ID|Status|
|---|---|
|1|todo|
|2|in progress|
|3|done|

**TASKS**
|Task ID|Account ID|Details|Created At|Status ID|
|---|---|---|---|---|
|1|2|Buy pencils.|2019-05-06 17:40:03|2|
|2|3|Buy books.|2019-05-07 7:40:03|1|

Finally, we also have another requirement to store account session data. We shall store it as shown in below table:

**ACCOUNT_SESSIONS**
|Session ID|Account ID|Session Created|Session End|
|---|---|---|---|
|asd1gh|1|2019-05-06 17:40:03|2019-05-06 18:00:03|

Normally, in enterprise applications IDs are not stored as integers. Because it will be easier for someone to query others information by just using an integer! In real world applications, IDs are not numeric but alphanumeric, with upto 100 characters. Thus, making it impossible for someone to guess another ID!

## Application Architecture
We shall develop this application following the famous and widely used MVC 2 desgin pattern. Our application will be action based. When a user sends a HTTP request to our application, we translate it to corresponding action on our application. The actions we support are create, read and update (CRU). Our application essentially is a data driven. It facilitates actions on the database. It helps users to store and manage their data on a remote database securely and safely with the help of authentication and authorization mechanisms. It acts as an HTML and HTTP based interface to the database.

When a user makes a HTTP request to our application, we send back requested data in the form of HTML. HTML supports links and forms to help users interact with web applications. Links are used to retrieve/get (HTTP GET) information, while forms are used to post data (HTTP POST) to the web application.

So, here's how we are going to translate HTTP requests to actions:

| HTML Element | HTTP Method   | Application Action  |
| :------------ |:-------------|:---------|
| Hyperlink | HTTP GET      | Read details |
| Form | HTTP POST     | Create or update |

HTTP GET sends the data as query parameters to the URL. While, HTTP POST sends the data in HTTP request body. HTTP POST doesn't reveal the data through the URL, whereas HTTP GET does. So, HTTP GET is not suitable for sending login credentials. Nobody wants to see their username and password appended to the HTTP request URL! We shall use HTTP POST to send user's credentials while logging in.

So, far we have decided how we are going to use HTTP, HTML and a database. There's another concept of HTTP that's essential for us to understand to decide the architecture of our HTTP based application. That's URL - uniform resource locator. Here's an example URL of a web application hosted on example.com server:

http://www.example.com/webapp/details?id=12




## Class Diagrams

## Data Layer

## Controllers

## Views
