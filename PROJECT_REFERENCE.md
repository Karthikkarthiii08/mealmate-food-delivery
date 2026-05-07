# MealMate – Project Reference

## Tech Stack
- Backend: Django 5.1.4 (Python)
- Database: SQLite3 (db.sqlite3)
- Payment: Razorpay
- Frontend: HTML + CSS (inline, no framework)
- Theme: CSS custom properties (light/dark toggle via localStorage)

---

## Project Structure

```
Mealmate/
├── manage.py
├── db.sqlite3
├── requirements.txt
├── meal_buddy/                  ← Django project config
│   ├── settings.py
│   ├── urls.py
│   ├── wsgi.py
│   └── asgi.py
└── delivery/                    ← Main app
    ├── models.py
    ├── views.py
    ├── urls.py
    ├── admin.py
    ├── static/
    │   └── delivery/
    │       └── theme.css        ← Shared CSS variables (light/dark theme)
    └── templates/
        └── delivery/
            ├── index.html
            ├── signin.html
            ├── signup.html
            ├── admin_home.html
            ├── show_restaurants.html
            ├── add_restaurant.html
            ├── update_restaurant.html
            ├── update_menu.html
            ├── customer_home.html
            ├── customer_menu.html
            ├── cart.html
            ├── checkout.html
            ├── orders.html
            ├── success.html
            └── fail.html
```

---

## Database Models (delivery/models.py)

### Customer
| Field    | Type      | Notes              |
|----------|-----------|--------------------|
| username | CharField | max 20             |
| password | CharField | max 20             |
| email    | CharField | max 20             |
| mobile   | CharField | max 10             |
| address  | CharField | max 50             |

### Restaurant
| Field   | Type     | Notes              |
|---------|----------|--------------------|
| name    | CharField | max 20            |
| picture | URLField  | max 200, has default |
| cuisine | CharField | max 200           |
| rating  | FloatField |                  |

### Item
| Field       | Type       | Notes                          |
|-------------|------------|--------------------------------|
| restaurant  | ForeignKey | → Restaurant (CASCADE)         |
| name        | CharField  | max 20                         |
| description | CharField  | max 200                        |
| price       | FloatField |                                |
| vegeterian  | BooleanField | default False                |
| picture     | URLField   | max 400, has default           |

### Cart
| Field    | Type       | Notes                        |
|----------|------------|------------------------------|
| customer | ForeignKey | → Customer (CASCADE)         |
| items    | M2M        | → Item                       |
| total_price() | method | sum of all item prices      |

---

## URL Routes (delivery/urls.py)

| URL | View | Name | Description |
|-----|------|------|-------------|
| `/` | index | — | Landing page |
| `/open_signin` | open_signin | open_signin | Show sign in page |
| `/open_signup` | open_signup | open_signup | Show sign up page |
| `/signup` | signup | signup | Handle sign up POST |
| `/signin` | signin | signin | Handle sign in POST |
| `/open_add_restaurant` | open_add_restaurant | open_add_restaurant | Show add restaurant form |
| `/add_restaurant` | add_restaurant | add_restaurant | Handle add restaurant POST |
| `/open_show_restaurant` | open_show_restaurant | open_show_restaurant | List all restaurants (admin) |
| `/open_update_restaurant/<id>` | open_update_restaurant | open_update_restaurant | Show update form |
| `/update_restaurant/<id>` | update_restaurant | update_restaurant | Handle update POST |
| `/delete_restaurant/<id>` | delete_restaurant | delete_restaurant | Delete restaurant |
| `/open_update_menu/<id>` | open_update_menu | open_update_menu | Show menu management page |
| `/update_menu/<id>` | update_menu | update_menu | Add item to menu POST |
| `/delete_menu_item/<item_id>/<restaurant_id>` | delete_menu_item | delete_menu_item | Delete menu item |
| `/view_menu/<restaurant_id>/<username>` | view_menu | view_menu | Customer views menu |
| `/add_to_cart/<item_id>/<username>` | add_to_cart | add_to_cart | Add item to cart |
| `/remove_from_cart/<item_id>/<username>` | remove_from_cart | remove_from_cart | Remove item from cart |
| `/show_cart/<username>` | show_cart | show_cart | View cart |
| `/checkout/<username>/` | checkout | checkout | Razorpay checkout |
| `/orders/<username>/` | orders | orders | Order confirmation, clears cart |
| `/logout` | logout | logout | Redirect to index |

---

## Views Summary (delivery/views.py)

| View | What it does |
|------|-------------|
| index | Renders landing page |
| open_signin / open_signup | Renders auth forms |
| signup | Creates Customer, redirects to signin |
| signin | Validates credentials → admin_home or customer_home |
| add_restaurant | Creates Restaurant object |
| open_show_restaurant | Lists all restaurants |
| update_restaurant | Updates restaurant fields |
| delete_restaurant | Deletes restaurant |
| open_update_menu | Shows menu items for a restaurant |
| update_menu | Creates Item linked to restaurant, redirects back to menu |
| delete_menu_item | Deletes Item |
| view_menu | Shows items for a restaurant to customer |
| add_to_cart | Adds item to Cart M2M, redirects to menu |
| remove_from_cart | Removes item from Cart M2M |
| show_cart | Shows cart items + total |
| checkout | Creates Razorpay order, passes key/order_id to template |
| orders | Shows confirmation, clears cart |
| logout | Renders index page |

---

## Settings (meal_buddy/settings.py)

| Key | Value |
|-----|-------|
| DEBUG | True (local) |
| ALLOWED_HOSTS | ['*'] (local) |
| DATABASE | SQLite3 → db.sqlite3 |
| STATIC_URL | static/ |
| STATIC_ROOT | BASE_DIR/staticfiles |
| RAZORPAY_KEY_ID | rzp_test_SmK31H7WLbidcA |
| RAZORPAY_KEY_SECRET | (set in settings.py) |

---

## Static Files

| File | Purpose |
|------|---------|
| delivery/static/delivery/theme.css | CSS variables for light/dark theme, toggle button style |

### Theme Variables
- `--bg` — page background gradient
- `--surface` — card/panel background
- `--text-primary / secondary / muted` — text colors
- `--orange / --orange-dark` — brand accent color
- `--border / --border-light` — borders
- `--red-bg / --red` — delete/error colors
- `--green-bg / --green` — veg/success colors
- `--input-bg` — form input background

Theme is toggled via a 🌙/☀️ floating button and saved in `localStorage` as `mb-theme`.

---

## Admin Login
- Username: `admin`
- Password: whatever was set via `createsuperuser` or directly in Customer table

## How to Run Locally
```cmd
myenv\Scripts\activate
python manage.py migrate
python manage.py runserver
```
Open: http://127.0.0.1:8000

## Deploy (PythonAnywhere)
1. Upload project files
2. Create virtualenv, install requirements.txt
3. Set DEBUG=False, ALLOWED_HOSTS=['yourusername.pythonanywhere.com']
4. Run collectstatic
5. Configure WSGI file pointing to meal_buddy.settings
6. Reload web app
