# README

# Introduction

Todo - An easy sweet application for writing a daily to-do list.

## Table of Contents

1. Project Description
2. Installation
3. Usage
4. Diagram
5. Routes
6. Documentation
   1. Packages
   2. Schemas
   3. Tasks
   4. Authentication

## Project Description

Todo application is a kind of app generally used to maintain our day-to-day tasks or list everything we have to do, with log-in and log-out functionalities. Users can only add their tasks to their list after logging in. After they add the task, they can see a table that shows the task in order and time. Users can delete and update their tasks at any time. After the change, it will directly show in the form. When people push the log-out button, they will direct to the home page where they start from the begining. Nevertheless, the tasks they made that have yet to be deleted will stay on the list the next time they log in.

## Installation

1. Install `virtualenv`:

```other
$ pip3 install virtualenv
```

2. Open a terminal in the project root directory and run:

```other
$ virtualenv env
```

3. Then run the command:

```other
$ source env/bin/activate
```

4. Then install the dependencies:

```other
$ (env) pip3 install -r requirements.txt
```

5. Finally start the web server:

```other
$ (env) python3 app.py
```

This server will start on port 7979.

## Usage

#### Windows:

python [app.py](http://app.py)

#### macOS/Linux:

python3 [app.py](http://app.py)

## Diagram

![Flask-App_Diagramm.jpeg](https://res.craft.do/user/full/7abbab62-8b5c-faea-1b7e-730cd2f0f64d/doc/37E94F9C-6CE4-474C-A711-22BC65757E98/0788F519-0E14-4BF0-A010-005C13B808B4_2/e0kUuFyQ25S5lmwPLqUUBLx64kFOMG86ldiZBgWMvXIz/Flask-App_Diagramm.jpeg)

## Routes

The website consists of 5 main routes:

| Route       | Description                                                                                  |
| ----------- | -------------------------------------------------------------------------------------------- |
| /           | Shows todo’s logo and slogan. Here we have the option to login or register.                  |
| /register   | User can enter their data to create an account                                               |
| /login      | User can access data to login                                                                |
| /logout     | User can logout                                                                              |
| /addTask    | User sees an overview of all tasks, they can update and delete tasks; They can create a task |
| /update/:id | Route for updating a task queried by id                                                      |
| /delete/:id | Route for deleting a task queried by id                                                      |

## Documentation

### Packages

Flask: For injecting content into html

SQLAlchemy: For SQL Database

Flask_login: For Authentication

Flast_wtf: For Forms and Form Validation

Flask_bcrypt: For password encryption

### Schemas

For the database we set up the models User and Todo:

```other
class Todo(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    content = db.Column(db.String(200), nullable=False)
    date_created = db.Column(db.DateTime, default=datetime.utcnow)

    def __repr__(self):
        return '<Task %r>' % self.id

class User(db.Model, UserMixin):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(20),nullable=False, unique=True)
    password = db.Column(db.String(20),nullable=False)
```

For the User model we utilize `flask_login` and add UserMixin, which gives us some functionality for authentication.

### Tasks

To show my understanding of flask and python, I will explain what happens in the addTask route:

```other
@app.route('/addTask',methods=['POST','GET'])
@login_required
def addTask():
    if request.method=='POST':
        task_content = request.form['content']
        if(task_content == ""):
            return 'Please fill out the text field'
        new_task = Todo(content=task_content)
        try:
            db.session.add(new_task)
            db.session.commit()
            return redirect('/addTask')
        except:
            return 'There was an issue adding your task.'

    else:
        tasks = Todo.query.order_by(Todo.date_created).all()
        return render_template('addTask.html', tasks=tasks)
```

First we define the route with `@app.route`. We set it to `/addTask`. Then we define the allowed methods `POST, GET`. We also require the user to be logged in `@login_required`.

Then we check if the user is calling the route with the method POST. If this is not the case, such as when the user isn’t adding a task, we get All Tasks, ordered by creation date:

`tasks = Todo.query.order_by(Todo.date_created).all()`

and return “addTask.html”, giving the tasks to the page.

When the user enters a task and clicks on the button to add it:

```other
<form action="{{url_for('addTask')}}" method="POST">
        <textarea type="text" name="content" id="content-field" min="2" required></textarea>
        <input class="default-button" type="submit" value="Add">
</form>
```

We go to the route `/addTask` with the method `POST`. The app then extracts the taks text from the form field `task_content = request.form['content`’`]`, creates the new Task, and tries to add it to the database. If not succesfull at any step, it will show an error message.

### Authentication

Here I will explain the code for registering a user, with the help of `flask_login`:

```other
@app.route('/register', methods=['GET','POST'])
def register():
    form = RegisterForm()
    if request.method == "POST":
        if form.validate_on_submit():
            try:
                hashed_password = bcrypt.generate_password_hash(form.password.data)
                new_user = User(username=form.username.data, password=hashed_password)
                db.session.add(new_user)
                db.session.commit()
            except:
                flash("Something went wrong", "error")
            return redirect(url_for('login'))
        else:
            flash("Sorry, this username already exists, please try a new one.", "error")
    return render_template('register.html', form=form)
```

I will skip the parts explained in the section before. First we create a form object with the help of flask_wtf `form = RegisterForm()`. We then validate the submitted form with the help of the form object. We generate a hashed password with bcrypt: `bcrypt.generate_password_hash(form.password.data)`. Create a user from the hashed password and username and add it to the db.

We later use the user data to compare the entered data in the login form and check if a user has entered the right data.

