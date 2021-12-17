# L.U.N.A.L.A

Hosted on https://thechincheachilla.github.io/L.U.N.A.L.A/ \
Note: Does not work on the web; flask application currently not hosted

### Setup Instructions
A detailed instructional guide can be found here: https://testdriven.io/blog/developing-a-single-page-app-with-flask-and-vuejs/ \
The following commands assume that you are using a GIT terminal. I suggest GIT bash here: https://git-scm.com/downloads

### 1. Flask Setup
Note: PIP must be installed. If not, install PIP from Python; it comes included: https://www.python.org/downloads/
Navigate to the server directory \
Run: ```python -m venv env```\
Run: ```source env/bin/activate``` \
Run: ```pip install Flask==1.1.2 Flask-Cors==3.0.10```\
Run: ```pip install pyopenssl```

### 2. Additional Required imports
To install Pandas, navigate to the server directory \
Run: ```pip install pandas```\
To install most of the Vue dependenvies, navigate to the lunala directory \
Run: ```npm install```\
To install Axios, navigate to the lunala directory \
Run: ```npm install axios ```

### 3. Run the API Flask app
Navigate to the server directory and repeat the following flask setup steps: \
Run: ```python -m venv env```\
Run: ```source env/bin/activate``` \
Navigate to the env directory \
Run: ```python app.py```\
Hosted on http://127.0.0.1:5000/

### 4. Run the Frontend Vue app
Navigate to the lunala directory
Run: ```npm sun serve```
Head to http://localhost:8080/

### Deployment on GH-Pages
Run: ```npm run-script deploy```\
Navigate to GH-Pages, ensure correct branch is selected \
Currently hosted on https://thechincheachilla.github.io/Lunala/


## VUE Project setup
```
npm install
```

### Compiles and hot-reloads for development
```
npm run serve
```

### Compiles and minifies for production
```
npm run-script build
```

### Lints and fixes files
```
npm run lint
```

### Customize configuration
See [Configuration Reference](https://cli.vuejs.org/config/).
    
