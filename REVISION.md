# 3rd Year Programming – Exam Revision (Flask · Jinja · Bootstrap · HTML · Python)

<a id="top"></a>

## Contents

- [0. Setup – Flask, venv, running (from `Poznámky-1.pdf`)](#s0)
- [0.5 Key imports (one table)](#s0_5)
- [1. Flask basics](#s1)
- [2. Routing (`exercises/app_2.py`)](#s2)
- [3. Templates – `render_template`](#s3)
- [4. Jinja2 template syntax](#s4)
- [5. HTML fundamentals (everything used in your templates)](#s5)
- [6. Bootstrap 5](#s6)
- [7. Python concepts used](#s7)
- [8. External APIs – `requests` (project `6 api`)](#s8)
- [9. Project walkthroughs (what each page shows)](#s9)
- [10. Request flow – explain it in one breath](#s10)
- [11. Mistakes, weak spots and assignment mismatches (know these!)](#s11)
- [12. Quick-fire cheat sheet](#s12)
- [Appendix – key excerpts from your code](#excerpts)
  - [Project 5 – preparing data in Python (computed keys, if/elif, average)](#x1)
  - [Project 5 – cards with conditional class + badge](#x2)
  - [Project 5 – alerts and progress bars built from data](#x3)
  - [Project 4 – `zip` in Python → tuple unpacking in Jinja](#x4)
  - [Project 4 – list of dicts → inline if, `format` filter, `for … else`](#x5)
  - [Project 6 – API response → template](#x6)


Everything here comes from the code and teacher materials in `flask/`:

| File | What it teaches |
|---|---|
| `docs/Poznámky-1.pdf` | What Flask is, venv, installing, running, debug mode (section 0) |
| `docs/Cvičenia.pdf` + `exercises/hello_1.py` | First Flask app, routes, returning raw HTML strings |
| `exercises/app_2.py` | Dynamic routes, URL converters (`int`, `path`), `escape`, `url_for`, `test_request_context` |
| `projects/3_template_flask_(jinja2_bootstrap)/` | `render_template`, template inheritance (`base.html`), Bootstrap **navbar**, table with `loop.index` |
| `projects/4_templates_flask/` (+ `Cvičenia-2.pdf`) | Passing variables, `zip`, tuple unpacking in Jinja, `if/else`, `for … else`, inline `if`, `format` filter |
| `projects/5_flask/` (+ `Cvičenia - časť 1.pdf`) | List of dicts, computing data in Python, **cards, badges, alerts, progress bars**, `length` filter |
| `projects/6 api/` (+ `Cvičenia - časť 2.docx`) | Calling an external **API** with `requests`, `.json()`, showing API data in templates |

- [**Section 0**](#s0) = setup (venv, running the app).
- [**Section 0.5**](#s0_5) = all the key imports in one table.
- [**Section 8**](#s8) = APIs (`requests`).
- [**Section 11**](#s11) = bugs, weak spots and places where your code doesn't match the assignment sheet. Read it, because teachers love asking "what's wrong here?".

---

<a id="s0"></a>

## 0. Setup – Flask, venv, running (from `Poznámky-1.pdf`)

### 0.1 What Flask is

- A **web framework** written in Python for building web apps (backend: handles requests, talks to databases, generates dynamic pages).
- **Lightweight and flexible**: gives you the basic tools and leaves the details to you. It scales from small to bigger apps. Used by e.g. Pinterest and LinkedIn.
- **Alternative: Django** (also Python), better suited to large projects ("batteries included").
- Requires **Python 3.8+**.

**Libraries installed automatically with Flask** (likely theory question):

| Library | Role |
|---|---|
| **Werkzeug** | implements **WSGI**, the standard interface between Python apps and web servers |
| **Jinja** | the template language → dynamic HTML pages |
| **MarkupSafe** | safe insertion of data into HTML (prevents injection attacks). This is where `escape` comes from |
| **ItsDangerous** | securely signs data (e.g. session cookies) |
| **Click** | command-line tools (the `flask` command itself) |
| **Blinker** | support for signals |

⚠ `requests` is **NOT** part of Flask. It needs its own `pip install requests` (project 6).

### 0.2 Virtual environment (venv)

**Why:** it isolates each project's libraries, so updating a library in one project doesn't break another. Different projects can use different library or Python versions.

Windows (cmd), as in the notes:

```bat
> mkdir myproject                 :: make directory
> cd myproject                    :: change directory
> py -3 -m venv .venv             :: create venv in folder .venv
> .venv\Scripts\activate          :: activate it – prompt now shows (.venv)
> pip install Flask               :: install Flask INTO the venv
```

(macOS/Linux: `python3 -m venv .venv` and `source .venv/bin/activate`.)

### 0.3 Running the app

```bat
> flask --app hello run            :: runs hello.py  (also: python -m flask --app hello run)
> flask --app hello run --debug    :: debug mode
```

Output:

```
* Serving Flask app 'hello'
* Running on http://127.0.0.1:5000 (Press CTRL+C to quit)
```

- Open `http://127.0.0.1:5000` in the browser. **CTRL+C** stops the server.
- `--app hello` = the file name **without** `.py`. If the file is called `app.py` or `wsgi.py`, a plain `flask run` finds it automatically.
- **Never name your file `flask.py`.** It would clash with the Flask library itself, so `from flask import Flask` would import your own file.

**Debug mode** (`--debug`):

1. The server **auto-restarts** when you change code.
2. On an error you get an **interactive debugger in the browser** with details.
3. ⚠ **Never use it in production** (on a public server), because it lets anyone run code from the browser.

Alternative: put `app.run(debug=True)` inside `if __name__ == "__main__":` and run `python main.py` (see 1.2).

---

<a id="s0_5"></a>

## 0.5 Key imports (one table)

```python
from flask import Flask, render_template, url_for, request, redirect
from markupsafe import escape
import requests as req
from datetime import datetime
```

| Import | From | Used in | Why |
|---|---|---|---|
| `Flask` | `flask` | every app | the class you create the app from: `app = Flask(__name__)` |
| `render_template` | `flask` | projects 3–6 | renders an HTML file from `templates/` with variables |
| `url_for` | `flask` | `app_2.py` (in Python). Templates have it automatically | builds a URL from a **function name** |
| `escape` | `markupsafe` | `app_2.py` | makes user text safe to put into HTML (stops XSS) |
| `requests` (as `req`) | separate package | project 6 | sends HTTP requests to external APIs |
| `request` | `flask` | *(not in your code)* | data from the **incoming** request: form fields, query string, method |
| `redirect` | `flask` | *(not in your code)* | sends the browser to another URL: `return redirect(url_for('index'))` |
| `datetime` | `datetime` (standard library) | *(the optional `/time` task)* | current date/time |

Correcting three common mix-ups:

- **`escape` is not "for dynamic routes".** It's for *any* time you put user-controlled text into HTML **yourself** (returning a raw string). Dynamic routes are just the most common place that happens, because the URL is user input. Inside templates you don't need it, because Jinja auto-escapes.
- **`escape` comes from `markupsafe`, not `flask`.** Old tutorials show `from flask import escape`. It was deprecated and **removed in Flask 3.0**, so it now raises `ImportError`.
- **`request` ≠ `requests`.** `flask.request` = the request **coming in** to your app. `requests` = a library your app uses to send requests **out** to another server. One letter apart, different things.

---

<a id="s1"></a>

## 1. Flask basics

### 1.1 Minimal app (`exercises/hello_1.py`)

```python
from flask import Flask

app = Flask(__name__)          # create the app; __name__ tells Flask where to find templates/static

@app.route("/")                # decorator: URL "/" → this function
def hello_world():
    return "<H1>Hello, World!</H1>" "<p>Vitajte...</p>" "<p>lorem ipsum</p>"

@app.route("/test_stranky")
def lorem():
    return "<p>cauahojcaudovidenia</p>"
```

Key points:

- `Flask(__name__)` – creates the application object. `__name__` is the module name (`"__main__"` when run directly, `"hello"` when imported).
- `@app.route("/url")` – a **decorator**; it registers the function below it as the handler (a "view function") for that URL.
- Whatever the view function **returns** is sent to the browser as the response body. A string is treated as HTML.
- `"a" "b" "c"` – Python **implicit string concatenation**: adjacent string literals are joined into one string (`"abc"`). That's why `hello_1.py` works without `+`.
- `/test_stranky` was the optional task from `Cvičenia.pdf`: a second `@app.route()` + a new function that returns text.
- Each function name must be unique — Flask uses the function name as the **endpoint** name (used by `url_for`).

### 1.2 Running the app

Full details are in section 0.3. None of your files has a run block, so you start them with `flask --app main run --debug` (for `main.py`).

Or add this to the bottom of the file and run `python main.py`:

```python
if __name__ == "__main__":
    app.run(debug=True)
```

Default address: `http://127.0.0.1:5000/`.

### 1.3 Project structure (projects 3–6)

```
project/
├── .venv/              ← virtual environment (don't edit, don't submit)
├── main.py  (or app.py)
├── templates/          ← MUST be called "templates" – render_template looks here
│   ├── base.html
│   └── index.html
└── static/             ← (optional) CSS, JS, images → url_for('static', filename='style.css')
```

`__pycache__/` is just Python's compiled bytecode cache. It's generated automatically, so ignore it.

---

<a id="s2"></a>

## 2. Routing (`exercises/app_2.py`)

Imports for this file:

```python
from flask import Flask, url_for
from markupsafe import escape
```

### 2.1 Static routes

```python
@app.route("/")
def index():
    return "<H1>Vitaj na hlavnej stránke!</H1>"

@app.route("/about")
def about():
    return "<H1>Toto je stránka o nás.</H1>"

@app.route("/contact")
def contact():
    return "<H1>Kontaktujte nás na e-maile...</H1>"
```

### 2.2 Dynamic routes (variable parts)

```python
@app.route("/user/<username>")            # default converter = string
def profile(username):
    return f'User {escape(username)}'

@app.route('/post/<int:post_id>')         # only integers match; value arrives as int
def show_post(post_id):
    return f'Post {post_id}'

@app.route('/path/<path:subpath>')        # like string but ALSO accepts "/"
def path(subpath):
    return f'Podcesta: {escape(subpath)}'
```

- `<name>` in the URL → becomes a **function parameter with the same name**.
- Converters:

| Converter | Accepts | Example URL | Value |
|---|---|---|---|
| `string` (default) | any text **without** `/` | `/user/jano` | `"jano"` |
| `int` | positive integers | `/post/3` | `3` (int) |
| `float` | positive floats | `/x/1.5` | `1.5` |
| `path` | text **including** `/` | `/path/images/photos` | `"images/photos"` |
| `uuid` | UUID strings | | |

- `/post/abc` → **404**, because `abc` is not an int.

### 2.3 Trailing slash rule

```python
@app.route('/projects/')     # with trailing slash
@app.route('/login')         # without
```

- `/projects/` defined **with** slash → visiting `/projects` **redirects** to `/projects/` (behaves like a folder).
- `/login` defined **without** slash → visiting `/login/` gives **404 Not Found** (behaves like a file).

### 2.4 `escape()` – security (XSS)

```python
from markupsafe import escape
return f'User {escape(username)}'
```

When you return a raw string built from user input, someone could put `<script>…</script>` in the URL. `escape()` converts `<` → `&lt;`, `>` → `&gt;`, etc., so it is shown as text, not executed. This attack is called **XSS (Cross-Site Scripting)**.
In templates (`render_template`) you don't need it — **Jinja auto-escapes** `{{ }}` output in `.html` files.

### 2.5 `url_for()` – building URLs from function names

```python
from flask import url_for

with app.test_request_context():
    print(url_for("index"))                                 # /
    print(url_for("login"))                                 # /login
    print(url_for("profile", username='Palo Scerba'))       # /user/Palo%20Scerba
    print(url_for("show_post", post_id=3))                  # /post/3
    print(url_for("path", subpath="images/photos"))         # /path/images/photos
```

- First argument = **function name** (endpoint), **not** the URL.
- Keyword arguments fill in the `<variables>` of the route.
- Spaces and special characters get URL-encoded (`" "` → `%20`).
- Extra arguments that aren't in the route become a query string: `url_for("login", next="/x")` → `/login?next=%2Fx`.
- **Why use it instead of hard-coding `"/about"`?** If you change the URL in `@app.route`, all links update automatically; it also handles escaping.
- `test_request_context()` – fakes a request so `url_for` works outside a real browser request (just for testing/printing).

---

<a id="s3"></a>

## 3. Templates – `render_template`

```python
from flask import Flask, render_template      # ← render_template must be imported, or you get NameError

app = Flask(__name__)

@app.route("/")
def index():
    return render_template("index.html")

@app.route("/ziaci")
def students():
    names = ["Janko", "Marienka", "Fero", "Katka"]
    return render_template("students.html", names=names)
```

- `render_template("file.html", key=value, ...)` – loads the file from `templates/`, fills it with the variables and returns the finished HTML.
- `names=names` → left side = **name inside the template**, right side = **Python variable**. They don't have to match (`render_template("x.html", jmena=names)` → use `{{ jmena }}` in the template).
- You can pass as many variables as you want (project 5 passes 6):

```python
return render_template("index.html", users=users, users_data=users_data,
                       messages=messages, stats=stats, average=average, summary=summary)
```

- Passing values from the URL into the template:

```python
@app.route("/welcome/<name>")
def welcome(name):
    return render_template("welcome.html", name=name)
```
```html
<h1>Welcome, {{ name }}!</h1>
```

- Passing a boolean:

```python
@app.route("/login")
def login():
    logged_in = False
    return render_template("login.html", logged_in=logged_in)
```

- Route URL vs. function name vs. template name are three **independent** things:
  `@app.route("/kontakt")` → `def contact()` → `render_template("kontakt.html")` → in templates you link with `url_for('contact')` (function name!).

---

<a id="s4"></a>

## 4. Jinja2 template syntax

### 4.1 The three delimiters

| Syntax | Meaning | Example |
|---|---|---|
| `{{ ... }}` | **print** a value/expression | `{{ name }}`, `{{ user.age }}` |
| `{% ... %}` | **statement** (logic: if, for, block, extends) | `{% for x in xs %}` |
| `{# ... #}` | comment (not sent to browser) | `{# TODO #}` |

### 4.2 Template inheritance – `extends` and `block`

**`base.html`** (the parent / layout):

```html
<!DOCTYPE html>
<html lang="sk">
<head>
    <meta charset="UTF-8">
    <title>{% block title %}Moja stránka{% endblock %}</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css" rel="stylesheet">
</head>
<body>
    <div class="container mt-5">
        {% block content %}{% endblock %}
    </div>
</body>
</html>
```

**`index.html`** (the child):

```html
{% extends "base.html" %}

{% block title %}Domov{% endblock %}

{% block content %}
    <h1 class="text-primary">Vitajte!</h1>
    <p class="lead">Toto je jednoduchá Flask stránka s Bootstrapom.</p>
{% endblock %}
```

How it works:

- `{% extends "base.html" %}` must be the **first** thing in the child template.
- `{% block name %}...{% endblock %}` in the parent = a **placeholder** with optional **default content** ("Moja stránka" is shown if the child doesn't override `title`).
- The child defines blocks with the **same name** to replace them.
- Anything in the child **outside** a block is ignored.
- Why: write the `<head>`, Bootstrap link and navbar **once** (DRY – Don't Repeat Yourself); every page only writes its own content.
- `{{ super() }}` inside a child block would include the parent's default content too (not used in your code, but good to know).

### 4.3 Accessing data

```html
{{ name }}            {# simple variable #}
{{ user.name }}       {# dict key OR object attribute – Jinja allows dot syntax for dicts #}
{{ user['name'] }}    {# same thing, Python-style #}
{{ names[0] }}        {# list index #}
```

In Python you **must** write `user["name"]`; in Jinja `user.name` works too.

### 4.4 `for` loop

> **In context:** [P4 zip → table](#x4) · [P4 loop table](#x5)

```html
{% for name in names %}
    <tr>
        <th scope="row">{{ loop.index }}</th>
        <td>{{ name }}</td>
    </tr>
{% endfor %}
```

**The `loop` variable** (only exists inside a `for`):

| Variable | Value |
|---|---|
| `loop.index` | current iteration, **starting at 1** (used in your tables for `#`) |
| `loop.index0` | starting at 0 |
| `loop.first` | `True` on first iteration |
| `loop.last` | `True` on last iteration |
| `loop.length` | total number of items |
| `loop.revindex` | iterations remaining (ends at 1) |

**Tuple unpacking** (from `about.html`) – each item of `owners` is a tuple `(name, email, phone)`:

```html
{% for name, email, phone in owners %}
    <tr>
        <th scope="row">{{ loop.index }}</th>
        <td>{{ name }}</td>
        <td>{{ email }}</td>
        <td>{{ phone }}</td>
    </tr>
{% endfor %}
```

**`for … else`** (from `loop.html`) – the `else` part runs when the list is **empty**:

```html
{% for item in instruments %}
    <tr>...</tr>
{% else %}
    <tr><td colspan="3">Žiadne inštrumenty</td></tr>
{% endfor %}
```

⚠ Different from Python's `for…else` (in Python, `else` runs when the loop finishes without `break`). In Jinja it means "list was empty".

**Looping a list of dicts:**

```html
{% for user in users %}
<tr>
    <th>{{ user.name }}</th>
    <td>{{ user.age }}</td>
    <td>{{ user.city }}</td>
</tr>
{% endfor %}
```

### 4.5 `if / elif / else`

> **In context:** [P5 cards](#x2)

Block form (`login.html`):

```html
{% if logged_in %}
    <h1>Vitaj späť, používateľ!</h1>
{% else %}
    <h1>Prosím, prihláste sa.</h1>
{% endif %}
```

`{% elif condition %}` also exists. Every `if` must be closed with `{% endif %}`, every `for` with `{% endfor %}`, every `block` with `{% endblock %}`.

`if` inside an HTML attribute (adds a class conditionally – project 5):

```html
<div class="card h-100 {% if user.adult %}border-success{% endif %}">
```

if/else to show different elements:

```html
{% if user.adult %}
    <span class="badge bg-success">Plnoletý</span>
{% else %}
    <span class="badge bg-secondary">Neplnoletý</span>
{% endif %}
```

**Inline if (ternary expression)** inside `{{ }}` (`loop.html`):

```html
<tr class="{{ 'table-success' if item.change > 0 else 'table-danger' }}">
```

Syntax: `{{ A if condition else B }}` – same as Python.

### 4.6 Filters – `value|filter`

> **In context:** [P4 loop table](#x5)

A filter transforms a value, written with a pipe `|`.

| Filter | Example | Result |
|---|---|---|
| `length` | `{{ users_data|length }}` | `4` (number of items) |
| `format` | `{{ "%+.2f"|format(0.84) }}` | `+0.84` |
| `upper` / `lower` | `{{ name|upper }}` | `JANKO` |
| `title` / `capitalize` | `{{ "jan novak"|title }}` | `Jan Novak` |
| `default` | `{{ x|default("N/A") }}` | `N/A` if x undefined |
| `round` | `{{ 3.456|round(1) }}` | `3.5` |
| `join` | `{{ names|join(", ") }}` | `Janko, Marienka, ...` |
| `safe` | `{{ html|safe }}` | disables auto-escaping (dangerous with user input) |

**`format` string explained** – `"%+.2f"`:

- `%` start of placeholder
- `+` always show the sign (`+0.84`, `-1.23`)
- `.2` two decimal places
- `f` float

### 4.7 Dynamic values inside attributes

> **In context:** [P5 cards](#x2) · [P5 alerts + progress](#x3)

You can put `{{ }}` anywhere in HTML, including attributes, class names and inline styles:

```html
<img src="{{ user.img }}" alt="{{ user.name }}">
<div class="alert alert-{{ message.type }}">{{ message.text }}</div>   {# → alert-success, alert-danger... #}
<div class="progress-bar bg-{{ stat.color }}" style="width: {{ stat.progress }}%">
```

This is the key trick in project 5: Python decides `"success"`/`"danger"`, the template glues it onto the Bootstrap class prefix.

### 4.8 `url_for` in templates

```html
<a href="{{ url_for('index') }}">Domov</a>
<a href="{{ url_for('about') }}" class="btn btn-success">O nás</a>
<a href="{{ url_for('welcome', name='Jano') }}">Welcome</a>      {# → /welcome/Jano #}
<link rel="stylesheet" href="{{ url_for('static', filename='style.css') }}">
```

`url_for` is available in every template automatically — no import needed.

### 4.9 Auto-escaping

`{{ name }}` in a `.html` template is escaped automatically, so `/welcome/<script>alert(1)</script>` prints the tags as text. That's why `welcome.html` is safe without `escape()`.

---

<a id="s5"></a>

## 5. HTML fundamentals (everything used in your templates)

### 5.1 Document skeleton

```html
<!DOCTYPE html>                       <!-- tells browser: HTML5 -->
<html lang="sk">                      <!-- root element; lang = page language -->
<head>                                <!-- metadata, not visible -->
    <meta charset="UTF-8">            <!-- encoding – needed for č, ž, á... -->
    <title>Moja stránka</title>       <!-- text in browser tab -->
    <link href="...bootstrap.min.css" rel="stylesheet">   <!-- external CSS -->
</head>
<body>                                <!-- everything visible -->
    ...
</body>
</html>
```

Often also added (not in your files, but standard with Bootstrap — needed for proper mobile scaling):
`<meta name="viewport" content="width=device-width, initial-scale=1">`

### 5.2 Text elements

| Tag | Use |
|---|---|
| `<h1>` … `<h6>` | headings (h1 = biggest/most important; you use `h1` and `h5`) |
| `<p>` | paragraph |
| `<strong>` | bold, important text |
| `<em>` | italic / emphasis |
| `<span>` | inline container with no meaning (used for badges) |
| `<div>` | block container with no meaning (used for layout: container, card, alert…) |
| `<hr>` | horizontal line (self-closing) |
| `<br>` | line break (self-closing) |

**Block vs inline:** block elements (`div`, `p`, `h1`, `table`, `ul`) start on a new line and take the full width; inline elements (`span`, `a`, `strong`, `img`) sit inside text.

### 5.3 Links

```html
<a href="https://google.com">Google</a>        <!-- absolute URL -->
<a href="/about">About</a>                       <!-- relative to site -->
<a href="{{ url_for('about') }}">About</a>       <!-- Flask way -->
<a href="..." target="_blank">New tab</a>
```

`href` = where it goes. An `<a>` with Bootstrap class `btn` looks like a button.

### 5.4 Images

```html
<img src="{{ user.img }}" class="card-img-top" alt="{{ user.name }}">
```

- `src` – image URL. `alt` – alternative text (shown if image fails, read by screen readers). `<img>` has no closing tag.
- Size via attributes (project 6): `<img src="{{ dog_image.message }}" alt="Random Dog Image" height="400" width="400">`. The values are in pixels and need no `px`. Setting both can distort the image.
- Responsive with Bootstrap: `class="img-fluid"` (max-width 100%, keeps the aspect ratio). `rounded` gives rounded corners.

### 5.5 Lists

```html
<!-- Unordered (bullets) -->
<ul>
    <li>Safety</li>
    <li>Security</li>
    <li>Health</li>
</ul>

<!-- Ordered (numbers 1, 2, 3) -->
<ol>
    <li>First</li>
    <li>Second</li>
</ol>
```

- `<li>` = list item and must be inside `<ul>` or `<ol>` (your `4_templates_flask/templates/index.html` forgets the `<ul>`, see section 11).
- Dynamic list with Jinja:

```html
<ul>
    {% for name in names %}
        <li>{{ name }}</li>
    {% endfor %}
</ul>
```

- Bootstrap list version: `<ul class="list-group"><li class="list-group-item">…</li></ul>`.
- The navbar is also built from a `<ul>` (see 6.3).

### 5.6 Tables

```html
<table class="table table-striped table-hover">
    <thead>                                   <!-- header section -->
        <tr>                                  <!-- table row -->
            <th scope="col">#</th>            <!-- header cell (bold) -->
            <th scope="col">Meno</th>
        </tr>
    </thead>
    <tbody>                                   <!-- body section -->
        {% for name in names %}
            <tr>
                <th scope="row">{{ loop.index }}</th>   <!-- row header -->
                <td>{{ name }}</td>                     <!-- data cell -->
            </tr>
        {% endfor %}
    </tbody>
</table>
```

| Tag / attribute | Meaning |
|---|---|
| `<table>` | the table |
| `<thead>` / `<tbody>` | header / body groups |
| `<tr>` | **t**able **r**ow |
| `<th>` | **t**able **h**eader cell (bold, centered by default) |
| `<td>` | **t**able **d**ata cell |
| `scope="col"` | this header describes a column |
| `scope="row"` | this header describes a row |
| `colspan="3"` | cell spans 3 columns (used in the "Žiadne inštrumenty" row) |
| `rowspan="2"` | cell spans 2 rows |

### 5.7 Navigation

`<nav>` is a semantic element meaning "navigation links". See Bootstrap navbar in 6.3.

### 5.8 Attributes you used

| Attribute | Purpose |
|---|---|
| `class="..."` | CSS classes (Bootstrap); multiple separated by spaces |
| `style="width: 80%"` | inline CSS |
| `href` | link target |
| `src`, `alt` | image source and alt text |
| `rel="stylesheet"` | relationship of linked file (for `<link>`) |
| `lang`, `charset` | language, encoding |
| `scope`, `colspan` | table helpers |
| `id="..."` | unique identifier (not used but common) |

### 5.9 Forms (not in your code — but the `/login` route is begging for one, likely exam material)

```html
<form method="post" action="{{ url_for('login') }}">
    <div class="mb-3">
        <label for="user" class="form-label">Meno</label>
        <input type="text" class="form-control" id="user" name="username">
    </div>
    <div class="mb-3">
        <label for="pw" class="form-label">Heslo</label>
        <input type="password" class="form-control" id="pw" name="password">
    </div>
    <button type="submit" class="btn btn-primary">Prihlásiť</button>
</form>
```

```python
from flask import request

@app.route("/login", methods=["GET", "POST"])
def login():
    if request.method == "POST":
        username = request.form["username"]      # value from <input name="username">
        return render_template("welcome.html", name=username)
    return render_template("login.html")
```

- `methods=["GET","POST"]` – by default routes accept only GET.
- `request.form["name"]` – POST form data; `request.args.get("q")` – GET query string (`?q=...`).

---

<a id="s6"></a>

## 6. Bootstrap 5

### 6.1 Including Bootstrap

```html
<link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css" rel="stylesheet">
```

- Loaded from a **CDN** (Content Delivery Network) – no download needed.
- Goes in `<head>`, in `base.html` so every page gets it.
- This is **CSS only**. Interactive components (collapsing navbar hamburger, dropdowns, modals) also need the JS bundle before `</body>`:
  `<script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/js/bootstrap.bundle.min.js"></script>`

### 6.2 Layout – container

```html
<div class="container mt-5">
    {% block content %}{% endblock %}
</div>
```

- `container` – centered, fixed max-width box with side padding. (`container-fluid` = full width.)
- `mt-5` – margin-top size 5 (see spacing).

### 6.3 Navbar (`projects/3_template_flask_(jinja2_bootstrap)/templates/base.html`)

```html
<nav class="navbar navbar-expand-lg navbar-dark bg-primary">
    <div class="container">
        <ul class="navbar-nav">
            <li class="nav-item">
                <a class="nav-link" href="{{ url_for('index') }}">Domov</a>
            </li>
            <li class="nav-item">
                <a class="nav-link" href="{{ url_for('contact') }}">Kontakt</a>
            </li>
            <li class="nav-item">
                <a class="nav-link" href="{{ url_for('students') }}">Žiaci</a>
            </li>
        </ul>
    </div>
</nav>
```

| Class | Meaning |
|---|---|
| `navbar` | base navbar component |
| `navbar-expand-lg` | horizontal on **large** screens and up; vertical/stacked below |
| `navbar-dark` | **light text** (for a dark background) — confusing name! (In 5.3 the modern way is `data-bs-theme="dark"`) |
| `bg-primary` | blue background |
| `navbar-nav` | on the `<ul>` – the list of links |
| `nav-item` | on each `<li>` |
| `nav-link` | on each `<a>` |
| `container` | inside navbar – aligns links with page content |

Structure to remember: **nav → container → ul.navbar-nav → li.nav-item → a.nav-link**.

Full version with a brand name and mobile hamburger menu (needs Bootstrap JS):

```html
<nav class="navbar navbar-expand-lg navbar-dark bg-primary">
  <div class="container">
    <a class="navbar-brand" href="{{ url_for('index') }}">Moja stránka</a>
    <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#menu">
      <span class="navbar-toggler-icon"></span>
    </button>
    <div class="collapse navbar-collapse" id="menu">
      <ul class="navbar-nav">
        <li class="nav-item"><a class="nav-link active" href="#">Domov</a></li>
        <li class="nav-item"><a class="nav-link" href="#">Kontakt</a></li>
      </ul>
    </div>
  </div>
</nav>
```

`active` class = highlights the current page link.

### 6.4 Theme colours (used everywhere as suffixes)

| Name | Colour | Typical use |
|---|---|---|
| `primary` | blue | main |
| `secondary` | grey | neutral |
| `success` | green | OK / positive |
| `danger` | red | error / negative |
| `warning` | yellow | caution |
| `info` | light blue | information |
| `light` | light grey | |
| `dark` | almost black | |

They combine with prefixes: `text-*`, `bg-*`, `btn-*`, `alert-*`, `border-*`, `table-*`, `badge` + `bg-*`.

### 6.5 Typography / text utilities

| Class | Effect |
|---|---|
| `text-primary` | blue text (`text-danger` red, etc.) |
| `text-center` | center-aligned text (`text-start`, `text-end`) |
| `lead` | larger, lighter "intro" paragraph |
| `fw-bold` | font-weight bold (`fw-normal`, `fw-light`) |
| `fst-italic` | italic |

### 6.6 Spacing utilities

Format: **`{property}{side}-{size}`**

- property: `m` = margin (outside), `p` = padding (inside)
- side: `t` top, `b` bottom, `s` start(left), `e` end(right), `x` left+right, `y` top+bottom, *(none)* = all sides
- size: `0`–`5` (0, .25rem, .5rem, 1rem, 1.5rem, 3rem) or `auto`

Used in your code: `mt-5` (margin top big), `mb-1` (small margin bottom), `mb-3` (medium margin bottom), `g-3` (gutter = gap between grid columns).
Examples: `p-3`, `mx-auto` (center horizontally), `py-2`.

### 6.7 Buttons

```html
<a href="{{ url_for('index') }}" class="btn btn-success">Späť na domov</a>
```

- `btn` + `btn-{color}` – filled (`btn-primary`, `btn-danger`, …)
- `btn-outline-{color}` – outlined
- `btn-sm` / `btn-lg` – size
- Works on `<a>` and `<button>`.

### 6.8 Tables

| Class | Effect |
|---|---|
| `table` | basic Bootstrap table styling (**required**) |
| `table-striped` | zebra stripes on rows |
| `table-hover` | row highlights on mouse hover |
| `table-bordered` | borders on all cells |
| `table-dark` | dark table |
| `table-sm` | compact |
| `table-success` / `table-danger` / … | colour a **row** (`<tr>`) or cell — used in `loop.html` for +/− change |

Wrap in `<div class="table-responsive">` for horizontal scroll on phones.

### 6.9 Grid system

Bootstrap layout = **12 columns** in a **row** inside a **container**.

```html
<div class="container">
  <div class="row">
    <div class="col-md-8">wide</div>
    <div class="col-md-4">narrow</div>     <!-- 8 + 4 = 12 -->
  </div>
</div>
```

Breakpoints: `sm` ≥576px, `md` ≥768px, `lg` ≥992px, `xl` ≥1200px, `xxl` ≥1400px.
`col-md-4` = "4/12 width from medium screens up; full width below".

**row-cols** (project 5) – set the number of columns per row instead of each column's width:

```html
<div class="row row-cols-1 row-cols-md-4 g-3">
```

- `row-cols-1` – 1 item per row on small screens (phones)
- `row-cols-md-4` – 4 items per row on medium+ screens
- `g-3` – gutter (gap) of size 3 between items

### 6.10 Cards (project 5)

> **In context:** [P5 cards](#x2)

```html
<div class="row row-cols-1 row-cols-md-4 g-3">
    {% for user in users_data %}
    <div class="col">                                    <!-- ← correct pattern: wrap card in col -->
        <div class="card h-100 {% if user.adult %}border-success{% endif %}">
            <img src="{{ user.img }}" class="card-img-top" alt="{{ user.name }}">
            <div class="card-body">
                <h5 class="card-title">{{ user.name }}</h5>
                <p class="card-text mb-1">Mesto: {{ user.city }}</p>
                <p class="card-text">Vek: {{ user.age }}</p>
                {% if user.adult %}
                    <span class="badge bg-success">Plnoletý</span>
                {% else %}
                    <span class="badge bg-secondary">Neplnoletý</span>
                {% endif %}
            </div>
        </div>
    </div>
    {% endfor %}
</div>
```

| Class | Meaning |
|---|---|
| `card` | box with border & rounded corners |
| `card-img-top` | image at the top, fills card width |
| `card-body` | padded content area |
| `card-title` | title styling |
| `card-text` | body text |
| `h-100` | height 100% → all cards in a row have **equal height** |
| `border-success` | green border (here only if adult) |
| `card-header` / `card-footer` | optional top/bottom bars |

### 6.11 Badges

```html
<span class="badge bg-success">Plnoletý</span>
<span class="badge bg-secondary">Neplnoletý</span>
```

Small coloured label. `badge` + `bg-{color}`. `rounded-pill` makes it pill-shaped.

### 6.12 Alerts

> **In context:** [P5 alerts + progress](#x3)

```html
{% for message in messages %}
    <div class="alert alert-{{ message.type }}">{{ message.text }}</div>
{% endfor %}
```

`alert alert-{color}` – coloured message box. With `message.type = "danger"` → `alert alert-danger` (red).

### 6.13 Progress bars

> **In context:** [P5 alerts + progress](#x3)

```html
<p class="fw-bold mb-1">{{ stat.subject }}</p>
<div class="progress mb-3">                                  <!-- grey track -->
    <div class="progress-bar bg-{{ stat.color }}" style="width: {{ stat.progress }}%">
        {{ stat.progress }}%                                  <!-- label inside bar -->
    </div>
</div>
```

- Outer `progress` = background track; inner `progress-bar` = filled part.
- **Width is set via inline `style="width: X%"`** — that's what controls how full it is.
- Colour with `bg-success` / `bg-warning` / `bg-danger`. Extras: `progress-bar-striped`, `progress-bar-animated`.

---

<a id="s7"></a>

## 7. Python concepts used

### 7.1 Imports

```python
from flask import Flask, render_template, url_for   # import specific names from a package
from markupsafe import escape
import requests as req                              # import whole module under an alias
```

`from X import Y` → use `Y` directly. `import X as Z` → use `Z.something` (e.g. `req.get`). See section 0.5 for what each import does.

### 7.2 Functions & decorators

```python
@app.route("/")      # decorator – wraps/registers the function below
def index():         # function definition
    return "..."     # return value
```

A decorator is a function that takes another function and adds behaviour to it. `@app.route` adds the function to Flask's URL map.

### 7.3 f-strings

```python
return f'User {escape(username)}'
return f'Post {post_id}'
```

`f"...{expression}..."` inserts the value of the expression into the string.

### 7.4 Lists

```python
names = ["Janko", "Marienka", "Fero", "Katka"]
names[0]          # "Janko"
len(names)        # 4
names.append("X") # add item
```

### 7.5 Dictionaries & list of dictionaries

```python
user = {"name": "Anna", "age": 17, "city": "Bratislava"}
user["name"]          # "Anna"
user["adult"] = True  # add / change a key

users = [
    {"name": "Anna",  "age": 17, "city": "Bratislava"},
    {"name": "Marek", "age": 18, "city": "Košice"},
]
users[1]["city"]      # "Košice"
```

A list of dicts = like rows of a table, each dict = one row with named columns. Perfect for passing to a template and looping.

### 7.6 Modifying data in a loop (adding computed keys)

```python
for user in users_data:
    user["adult"] = user["age"] >= 18
```

- `user["age"] >= 18` is a comparison → evaluates to `True` or `False`, stored directly as the value.
- Dicts are **mutable** and the loop variable refers to the same dict, so changes stay in `users_data`.

### 7.7 if / elif / else

> **In context:** [P5 data prep](#x1)

```python
for stat in stats:
    if stat["progress"] >= 70:
        stat["color"] = "success"
    elif stat["progress"] >= 40:
        stat["color"] = "warning"
    else:
        stat["color"] = "danger"
```

Checked **top to bottom**, first true branch wins. Order matters: checking `>= 40` before `>= 70` would make 80 "warning".

Result with your data: Python 80 → success, HTML/CSS 60 → warning, Flask 40 → warning, SQL 20 → danger.

### 7.8 `sum`, `len`, `round`, generator expressions

> **In context:** [P5 data prep](#x1)

```python
average = round(sum(stat["progress"] for stat in stats) / len(stats), 1)
```

Step by step:

1. `stat["progress"] for stat in stats` – **generator expression** producing 80, 60, 40, 20
2. `sum(...)` → 200
3. `len(stats)` → 4
4. `200 / 4` → 50.0
5. `round(50.0, 1)` → 50.0 (1 decimal place)

Then:

```python
if average >= 70:
    summary = {"type": "success", "text": "Výborný pokrok!"}
elif average >= 40:
    summary = {"type": "warning", "text": "Dobré, ale ešte je čo zlepšiť."}   # ← 50 lands here
else:
    summary = {"type": "danger", "text": "Treba pridať."}
```

Equivalent list comprehension: `[s["progress"] for s in stats]` → `[80, 60, 40, 20]`.

### 7.9 `zip()` and `list()`

> **In context:** [P4 zip → table](#x4)

```python
names  = ["John Doe", "Lester", ...]
emails = ["johndoe@gmail.com", "lester@gmail.com", ...]
phones = ["+1 5056466130", "+1 5056466131", ...]
owners = list(zip(names, emails, phones))
# [("John Doe", "johndoe@gmail.com", "+1 5056466130"), ("Lester", ...), ...]
```

- `zip` pairs up items with the **same index** from several lists into tuples.
- Stops at the **shortest** list.
- `zip` returns a lazy iterator → `list()` turns it into a real list (so it can be looped more than once / have `|length`).
- Template unpacks it: `{% for name, email, phone in owners %}`.

### 7.10 Separation of concerns (the big idea in project 5)

> **In context:** [P5 data prep](#x1) · [P5 alerts + progress](#x3)

**Python (main.py) = logic**: compute `adult`, `color`, `average`, `summary`.
**Template (index.html) = presentation**: just display values and pick CSS classes.
Keep calculations out of templates where possible.

### 7.11 `with` statement

```python
with app.test_request_context():
    print(url_for("index"))
```

`with` opens a context, runs the block, then cleans up automatically (also used for files: `with open("f.txt") as f:`).

### 7.12 `datetime` (optional `/time` task from `Cvičenia-2.pdf`. You didn't do it, but it's an easy exam question)

```python
from datetime import datetime

@app.route("/time")
def time():
    now = datetime.now()                         # current date + time as a datetime object
    return render_template("time.html", now=now)
```
```html
<p>Dátum: {{ now.strftime("%d.%m.%Y") }}</p>    {# 08.10.2026 #}
<p>Čas: {{ now.strftime("%H:%M:%S") }}</p>      {# 21:05:33 #}
```

`strftime` codes: `%d` day, `%m` month, `%Y` 4-digit year, `%H` hour (24h), `%M` minutes, `%S` seconds.
(The sheet asks for "DD:MM:RRRR". That's a typo for DD.MM.RRRR, but `"%d:%m:%Y"` would produce exactly what it says.)

You can also format in Python and pass the string instead: `render_template("time.html", date=now.strftime("%d.%m.%Y"))`.

---

<a id="s8"></a>

## 8. External APIs – `requests` (project `6 api`)

### 8.1 Concepts

- **API** (Application Programming Interface): a program or website that makes its data available to **other programs**. Here it's a URL that returns data instead of an HTML page.
- **JSON**: the text format APIs return. It looks exactly like Python dicts and lists: `{"key": "value", "list": [1, 2]}`.
- **`requests`**: a Python library that sends HTTP requests from **your** code to another server. Install it separately: `pip install requests`.
- Your Flask app acts as a **client** of the API (it asks for data) and as a **server** for the browser (it serves the page) at the same time.

### 8.2 Your code

> **In context:** [P6 API → template](#x6)

```python
from flask import Flask, render_template
import requests as req                       # "as req" = alias, shorter name

app = Flask(__name__)

@app.route("/")
def joke():
    response = req.get("https://official-joke-api.appspot.com/random_joke")   # HTTP GET request
    return render_template("joke.html", joke=response.json())                  # .json() → Python dict

@app.route("/pes")
def dog():
    response = req.get("https://dog.ceo/api/breeds/image/random")
    return render_template("dog.html", dog_image=response.json())
```

What the APIs return (after `.json()`):

```python
# joke API
{"type": "general", "setup": "Why did the ...?", "punchline": "Because ...", "id": 123}

# dog API
{"message": "https://images.dog.ceo/breeds/husky/n02110185_1469.jpg", "status": "success"}
```

Templates read the keys with dot syntax:

```html
<h4 class="lead">{{ joke.setup }}</h4>
<h4 class="lead">{{ joke.punchline }}</h4>
<a href="/" class="btn btn-success">Další vtip</a>

<img src="{{ dog_image.message }}" alt="Random Dog Image" height="400" width="400">
<a href="/pes" class="btn btn-success">Další pes</a>
```

- **"Next" button:** it just links to the **same route** again. Every page load runs the view function again, which calls the API again and gets a new random joke or dog. No JavaScript is needed.
- For the dog, the key is confusingly called `message`, but it holds the **image URL**. You always have to look at the actual JSON to know the key names. That's exactly what `test.py` is for:

```python
import requests as req
print(req.get("https://dog.ceo/api/breeds/image/random").json())   # inspect the JSON structure in the terminal
```

### 8.3 Useful `requests` stuff (not in your code, but good to know)

```python
response = req.get(url, timeout=5)   # give up after 5 s instead of hanging forever
response.status_code                 # 200 = OK, 404 = not found, 500 = server error
response.ok                          # True if status code < 400
response.json()                      # body parsed from JSON → dict/list
response.text                        # body as a raw string
```

Safer version:

```python
@app.route("/")
def joke():
    try:
        response = req.get("https://official-joke-api.appspot.com/random_joke", timeout=5)
        response.raise_for_status()               # raises an exception on 4xx/5xx
        data = response.json()
    except req.RequestException:
        data = {"setup": "API nie je dostupné.", "punchline": ""}
    return render_template("joke.html", joke=data)
```

Without this, if the API is down your page crashes with a **500 Internal Server Error**.

The API that returns a list instead of a dict (the Cat API from the docx) needs an index: `response.json()[0]["url"]` in Python, or `{{ cat[0].url }}` in the template.

---

<a id="s9"></a>

## 9. Project walkthroughs (what each page shows)

### `exercises/`

- `hello_1.py`: `/` returns h1 + 2 paragraphs, `/test_stranky` returns a paragraph.
- `app_2.py`: routes and converters, then prints `url_for` results.

### `projects/3_template_flask_(jinja2_bootstrap)` – multi-page site with navbar

| URL | Function | Template | Shows |
|---|---|---|---|
| `/` | `index` | `index.html` | Welcome heading + lead paragraph + "Späť na domov" button (links to itself, see 11) |
| `/kontakt` | `contact` | `kontakt.html` | Contact heading + lead paragraph |
| `/ziaci` | `students` | `students.html` | Striped table of 4 names with numbers via `loop.index` |

All extend `base.html`, which has the blue navbar with `url_for` links.

### `projects/4_templates_flask`

> **In context:** [P4 zip → table](#x4) · [P4 loop table](#x5)

| URL | Shows | Concept |
|---|---|---|
| `/` | list of 5 words + button to About | `<li>`, `url_for` |
| `/about` | table of owners (name, email, phone) | `zip`, tuple unpacking, `loop.index` |
| `/welcome/<name>` | "Welcome, Jano!" | URL variable → template |
| `/login` | "Prosím, prihláste sa." (because `logged_in=False`) | `{% if %}` |
| `/loop` | instruments table, green rows for positive change, red for negative, `+0.84%` | inline if, `format` filter, `for…else`, `colspan` |

### `projects/5_flask` – one page, four sections

> **In context:** [P5 data prep](#x1) · [P5 cards](#x2) · [P5 alerts + progress](#x3)

1. **Table** of `users` (name, age, city).
2. **Cards** of `users_data` with photo, green border + "Plnoletý" badge if 18+, total count via `|length` → 4.
3. **Alerts** – 4 coloured messages from `messages`, count via `|length`.
4. **Progress bars** – colour decided in Python, average 50.0% → yellow "Dobré, ale ešte je čo zlepšiť." alert.

### `projects/6 api`

| URL | Function | Template | Shows |
|---|---|---|---|
| `/` | `joke` | `joke.html` | random joke (setup + punchline) + "next joke" button |
| `/pes` | `dog` | `dog.html` | random dog picture 400×400 + "next dog" button |

---

<a id="s10"></a>

## 10. Request flow – explain it in one breath

1. Browser requests `http://127.0.0.1:5000/ziaci`.
2. Flask matches the URL to `@app.route("/ziaci")` → calls `students()`.
3. The function prepares data (`names`) and calls `render_template("students.html", names=names)`.
4. Jinja loads `students.html`, sees `{% extends "base.html" %}`, loads the base, replaces `title` and `content` blocks, runs the `for` loop, inserts values with `{{ }}` (auto-escaped).
5. Finished plain HTML is returned to the browser.
6. Browser downloads Bootstrap CSS from the CDN and renders the styled page.

The browser **never sees Jinja code** — only the final HTML (check with "View page source").

---

<a id="s11"></a>

## 11. Mistakes, weak spots and assignment mismatches (know these!)

1. **`<li>` without `<ul>`** – `4_templates_flask/templates/index.html` puts `<li>` directly in the content. Invalid HTML. Fix: wrap in `<ul>…</ul>`.
2. **Cards not wrapped in `.col`** – in project 5 the `.card` is a direct child of `.row`. Bootstrap's grid expects `row > col > card`; otherwise the gutter padding ends up *inside* the card's border and cards touch each other. Fix shown in 6.10.
3. **`<p></p>` used as a spacer** – empty paragraphs for spacing is a hack. Use spacing utilities (`mt-4`, `mb-5`) instead.
4. **Zero change is red** – in `loop.html`, `'table-success' if item.change > 0 else 'table-danger'` makes `0.00` red. If asked, a neutral case needs `{% if %}/{% elif %}` or a nested ternary.
5. **Navbar without brand/toggler/JS** – works, but there is no hamburger on mobile and no `navbar-brand`. Fine for school, but know the full version (6.3).
6. **No `if __name__ == "__main__": app.run()`** – you must run with `flask --app ... run`.
7. **`users` and `users_data` duplicate data** in project 5 – same people twice. Could use one list.
8. **No viewport meta tag** – Bootstrap's responsive classes (`row-cols-md-4`, `navbar-expand-lg`) behave badly on real phones without `<meta name="viewport" content="width=device-width, initial-scale=1">`.
9. **Uppercase `<H1>`** in `hello_1.py`/`app_2.py` – HTML is case-insensitive so it works, but lowercase is the convention.
10. **Hard-coded links in project 6**: `href="/"` and `href="/pes"` instead of `url_for('joke')` / `url_for('dog')`. That's exactly what section 2.5 says not to do. It works, but you'd lose marks if url_for is required.
11. **No error handling or timeout on API calls (project 6).** If the API is down or slow, the page crashes (500) or hangs. See 8.3.
12. **"Další" is Czech.** In Slovak it's **"Ďalší"** (in both project 6 buttons).
13. **`<h4 class="lead">`** in `joke.html`: `lead` is meant for paragraphs, and two h4s for a setup and a punchline aren't semantically headings. A `<p class="lead">` would be cleaner. Minor.
14. **`height="400" width="400"`** on the dog image forces a square and **distorts** non-square photos. Use only one dimension, or Bootstrap's `img-fluid` / CSS `object-fit: cover`.
15. **Project 3 `index.html` has a "Späť na domov" button that links to the home page itself**, which is pointless. Meanwhile `kontakt.html` has **no** way back except the navbar. The button belongs on `kontakt.html`.

**Checked against the assignment sheets:**

- **`Cvičenia-2.pdf`, task 3:** "index.html must contain at least one `<h1>` and **at least 5 paragraphs**". You have five `<li>` items, not five `<p>`, and without a `<ul>`. That doesn't meet the spec. Everything else on that sheet (about table with ≥5 owners and 3 columns, welcome/<name>, links home with `url_for`, login if/else, a Jinja loop page) is done. The optional `/time` task isn't done (see 7.12).
- **`Cvičenia - časť 2.docx`:** asks for the joke to be shown in **`index.html`**. You named it `joke.html`. That's functionally fine, but it's a literal mismatch if the teacher checks file names. The docx also says "knižnica request", but the library is **`requests`** (with an s).
- **`Cvičenia.pdf` and `Cvičenia - časť 1.pdf`:** fully done.

---

<a id="s12"></a>

## 12. Quick-fire cheat sheet

```python
# --- Setup ---
# py -3 -m venv .venv  →  .venv\Scripts\activate  →  pip install Flask requests
# flask --app main run --debug        (CTRL+C to stop, never --debug in production)

# --- Imports ---
from flask import Flask, render_template, url_for, request, redirect
from markupsafe import escape          # NOT from flask (removed in Flask 3)
import requests as req                 # external APIs (separate pip install)
from datetime import datetime

# --- API ---
data = req.get("https://...", timeout=5).json()     # → dict
return render_template("x.html", data=data)         # {{ data.key }}

# --- Flask ---
app = Flask(__name__)

@app.route("/")                         # static route
@app.route("/user/<name>")              # string var
@app.route("/post/<int:id>")            # int var
@app.route("/path/<path:p>")            # path var (allows /)
@app.route("/f", methods=["GET","POST"])

return "<h1>raw html</h1>"
return render_template("x.html", var=value)
url_for("function_name", param=value)
```

```html
<!-- --- Jinja --- -->
{% extends "base.html" %}
{% block content %} ... {% endblock %}
{{ variable }}   {{ dict.key }}   {{ list[0] }}
{% for x in xs %} {{ loop.index }} {% else %} empty {% endfor %}
{% for a, b, c in tuples %} ... {% endfor %}
{% if cond %} ... {% elif other %} ... {% else %} ... {% endif %}
{{ 'yes' if cond else 'no' }}
{{ xs|length }}   {{ "%+.2f"|format(n) }}   {{ s|upper }}
<a href="{{ url_for('index') }}">
{# comment #}
```

```html
<!-- --- Bootstrap --- -->
container  mt-5  mb-3  p-3  g-3  text-center  text-primary  lead  fw-bold
navbar navbar-expand-lg navbar-dark bg-primary > container > ul.navbar-nav > li.nav-item > a.nav-link
btn btn-success | btn-outline-primary | btn-lg
table table-striped table-hover | tr.table-success
row row-cols-1 row-cols-md-4 g-3 > col > card h-100 > img.card-img-top + card-body > card-title, card-text
badge bg-success
alert alert-danger
progress > progress-bar bg-warning style="width: 60%"
colours: primary secondary success danger warning info light dark
breakpoints: sm md lg xl xxl
```

---

<a id="excerpts"></a>

## Appendix – key excerpts from your code

Short pieces where it helps to see the Python side and the template side together. The **In context** links in the notes jump here.

<a id="x1"></a>

### Project 5 – preparing data in Python (computed keys, if/elif, average)

Python does all the deciding. It adds `adult`, picks a Bootstrap colour name for each stat, computes the average and chooses a summary. The template ([cards](#x2), [alerts + progress](#x3)) only displays the results.

`main.py` (5_flask)

```python
for user in users_data:
    user["adult"] = user["age"] >= 18
```

`main.py` (5_flask)

```python
for stat in stats:
    if stat["progress"] >= 70:
        stat["color"] = "success"
    elif stat["progress"] >= 40:
        stat["color"] = "warning"
    else:
        stat["color"] = "danger"
```

`main.py` (5_flask)

```python
average = round(sum(stat["progress"] for stat in stats) / len(stats), 1)

if average >= 70:
    summary = {"type": "success", "text": "Výborný pokrok!"}
elif average >= 40:
    summary = {"type": "warning", "text": "Dobré, ale ešte je čo zlepšiť."}
else:
    summary = {"type": "danger", "text": "Treba pridať."}

return render_template("index.html", users=users, users_data=users_data, messages=messages, stats=stats, average=average, summary=summary)
```

[↑ back to contents](#top)

---

<a id="x2"></a>

### Project 5 – cards with conditional class + badge

`adult` (computed in [P5 data prep](#x1)) is used twice: inside the `class` attribute for the green border, and in an if/else that picks the badge.

`templates/index.html` (5_flask)

```html
<div class="row row-cols-1 row-cols-md-4 g-3">
    {% for user in users_data %}
        <div class="card h-100 {% if user.adult %}border-success{% endif %}">
            <img src="{{ user.img }}" class="card-img-top" alt="{{ user.name }}">
            <div class="card-body">
                <h5 class="card-title">{{ user.name }}</h5>
                <p class="card-text mb-1">Mesto: {{ user.city }}</p>
                <p class="card-text">Vek: {{ user.age }}</p>
                {% if user.adult %}
                    <span class="badge bg-success">Plnoletý</span>
                {% else %}
                    <span class="badge bg-secondary">Neplnoletý</span>
                {% endif %}
            </div>
        </div>
    {% endfor %}
</div>
<p class="text-center">Celkový počet používateľov: <strong>{{ users_data|length }}</strong></p>
```

[↑ back to contents](#top)

---

<a id="x3"></a>

### Project 5 – alerts and progress bars built from data

`{{ }}` finishes Bootstrap class names (`alert-` + `success`) and the inline `width` style. The colour strings come from Python ([P5 data prep](#x1)).

`templates/index.html` (5_flask)

```html
{% for message in messages %}
    <div class="alert alert-{{ message.type }}">{{ message.text }}</div>
{% endfor %}
```

`templates/index.html` (5_flask)

```html
{% for stat in stats %}
    <p class="fw-bold mb-1">{{ stat.subject }}</p>
    <div class="progress mb-3">
        <div class="progress-bar bg-{{ stat.color }}" style="width: {{ stat.progress }}%">
            {{ stat.progress }}%
        </div>
    </div>
{% endfor %}
<hr>
<p class="text-center">Priemerný pokrok: <strong>{{ average }}%</strong></p>
<div class="alert alert-{{ summary.type }} text-center">{{ summary.text }}</div>
```

[↑ back to contents](#top)

---

<a id="x4"></a>

### Project 4 – `zip` in Python → tuple unpacking in Jinja

Three parallel lists become one list of tuples, and the template unpacks each tuple into three variables.

`app.py` (4_templates_flask)

```python
@app.route("/about") 
def about():
    names = ["John Doe", "Lester", "Rae", "Diddybop Brown", "Wilma"]
    emails = ["johndoe@gmail.com", "lester@gmail.com", "rae@gmail.com", "diddybop@gmail.com", "wilma@gmail.com"]
    telephone_numbers = ["+1 5056466130", "+1 5056466131", "+1 5056466132", "+1 5056466133", "+1 5056466134"]
    owners = list(zip(names, emails, telephone_numbers))
    return render_template("about.html", owners=owners)
```

`templates/about.html` (4_templates_flask)

```html
{% for name, email, phone in owners %}
    <tr>
        <th scope="row">{{ loop.index }}</th>
        <td>{{ name }}</td>
        <td>{{ email }}</td>
        <td>{{ phone }}</td>
    </tr>
{% endfor %}
```

[↑ back to contents](#top)

---

<a id="x5"></a>

### Project 4 – list of dicts → inline if, `format` filter, `for … else`

Row colour comes from an inline if. `%+.2f` adds the sign and 2 decimals. The `else` row only appears if the list is empty.

`app.py` (4_templates_flask)

```python
@app.route("/loop")
def loop():
    instruments = [
    {"symbol": "XAUUSD", "change": 0.84},
    {"symbol": "XAGUSD", "change": -1.23},
    {"symbol": "XPTUSD", "change": 2.56}]
    return render_template("loop.html", instruments=instruments)
```

`templates/loop.html` (4_templates_flask)

```html
{% for item in instruments %}
    <tr class="{{ 'table-success' if item.change > 0 else 'table-danger' }}">
        <th scope="row">{{ loop.index }}</th>
        <td>{{ item.symbol }}</td>
        <td>{{ "%+.2f"|format(item.change) }}%</td>
    </tr>
{% else %}
    <tr><td colspan="3">Žiadne inštrumenty</td></tr>
{% endfor %}
```

[↑ back to contents](#top)

---

<a id="x6"></a>

### Project 6 – API response → template

`req.get(...).json()` gives a dict, the dict goes into the template, and the keys are read with dot syntax.

`main.py` (6 api)

```python
@app.route("/") 
def joke():
    response = req.get("https://official-joke-api.appspot.com/random_joke")
    return render_template("joke.html", joke=response.json())
```

`templates/joke.html` (6 api)

```html
<h4 class="lead">{{ joke.setup }}</h4>
<h4 class="lead">{{ joke.punchline }}</h4>
```

[↑ back to contents](#top)

---

Good luck tomorrow!
