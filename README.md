"""
Flask-based single-file web app for a realistic, secure tourism website for Morocco (English).
Save this file as `app.py` in a project folder.

Features:
- Secure headers via Flask-Talisman (CSP, HSTS, Referrer-Policy)
- Simple contact form (input sanitized, rate-limited)
- Templates are created automatically on first run (no external scaffolding required)
- Static folder for images and assets (place your photos in `static/images/`)
- Production notes: use gunicorn/nginx or a cloud provider, enable HTTPS (Talisman forces HTTPS in production)

Requirements (put into requirements.txt):
Flask>=2.0
Flask-Talisman>=1.0
Flask-Limiter>=2.6
python-dotenv>=0.21

Run locally (development):
1. python -m venv venv && source venv/bin/activate (or venv\Scripts\activate on Windows)
2. pip install -r requirements.txt
3. export FLASK_APP=app.py
4. flask run --host=0.0.0.0 --port=5000

Production: use gunicorn + nginx (TLS handled by nginx) or deploy to a PaaS.

Note: The built-in Flask server is only for development. Talisman will try to enable HTTPS enforcement when the app is not running in debug mode.
"""

import os
import json
from pathlib import Path
from flask import Flask, render_template, request, redirect, url_for, flash
from markupsafe import escape
from flask_talisman import Talisman
from flask_limiter import Limiter
from flask_limiter.util import get_remote_address
from dotenv import load_dotenv

# Load .env if present
load_dotenv()

APP_DIR = Path(__file__).parent
TEMPLATES_DIR = APP_DIR / 'templates'
STATIC_DIR = APP_DIR / 'static'
IMAGES_DIR = STATIC_DIR / 'images'

# Create folder structure and sample files if missing
TEMPLATES_DIR.mkdir(exist_ok=True)
STATIC_DIR.mkdir(exist_ok=True)
IMAGES_DIR.mkdir(exist_ok=True)

# Write minimal templates if they don't exist
INDEX_HTML = TEMPLATES_DIR / 'index.html'
LAYOUT_HTML = TEMPLATES_DIR / 'layout.html'
GALLERY_HTML = TEMPLATES_DIR / 'gallery.html'
CONTACT_HTML = TEMPLATES_DIR / 'contact.html'
ABOUT_HTML = TEMPLATES_DIR / 'about.html'

if not LAYOUT_HTML.exists():
    LAYOUT_HTML.write_text('''<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>{{ title or 'Morocco Tourism' }}</title>
  <meta name="description" content="Explore Morocco — cities, deserts, coasts, culture and cuisine.">
  <link rel="stylesheet" href="/static/style.css">
</head>
<body>
  <header class="site-header">
    <div class="container">
      <h1><a href="{{ url_for('index') }}">Visit Morocco</a></h1>
      <nav>
        <a href="{{ url_for('index') }}">Home</a>
        <a href="{{ url_for('about') }}">About</a>
        <a href="{{ url_for('gallery') }}">Gallery</a>
        <a href="{{ url_for('contact') }}">Contact</a>
      </nav>
    </div>
  </header>
  <main class="container">
    {% with messages = get_flashed_messages() %}
      {% if messages %}
        <ul class="flashes">
        {% for m in messages %}
          <li>{{ m }}</li>
        {% endfor %}
        </ul>
      {% endif %}
    {% endwith %}
    {% block content %}{% endblock %}
  </main>
  <footer class="site-footer">
    <div class="container">&copy; {{ year }} Visit Morocco — Safe & Secure travel resources</div>
  </footer>
</body>
</html>''')

if not INDEX_HTML.exists():
    INDEX_HTML.write_text('''{% extends 'layout.html' %}
{% block content %}
<section>
  <h2>Welcome to Morocco 🇲🇦</h2>
  <p>From the red city of Marrakech to the blue lanes of Chefchaouen and the golden dunes of the Sahara, Morocco is a mosaic of landscapes and cultures.</p>
  <ul>
    <li>Historic medinas and UNESCO sites</li>
    <li>Coastal towns and surf destinations</li>
    <li>Desert excursions, mountain treks, and culinary tours</li>
  </ul>
  <p><a href="{{ url_for('gallery') }}">See photos</a> or <a href="{{ url_for('contact') }}">get in touch</a> for travel help.</p>
</section>
{% endblock %}''')

if not GALLERY_HTML.exists():
    GALLERY_HTML.write_text('''{% extends 'layout.html' %}
{% block content %}
<section>
  <h2>Gallery</h2>
  <div class="grid">
    {% for img in images %}
      <figure>
        <img src="{{ url_for('static', filename='images/' + img) }}" alt="{{ img }}" loading="lazy">
        <figcaption>{{ img.replace('-', ' ').rsplit('.',1)[0] }}</figcaption>
      </figure>
    {% endfor %}
  </div>
</section>
{% endblock %}''')

if not ABOUT_HTML.exists():
    ABOUT_HTML.write_text('''{% extends 'layout.html' %}
{% block content %}
<section>
  <h2>About Morocco</h2>
  <p>Morocco is a gateway between Africa and Europe, home to diverse landscapes and a rich cultural heritage. Travel safely by following local guidance and official travel advisories.</p>
</section>
{% endblock %}''')

if not CONTACT_HTML.exists():
    CONTACT_HTML.write_text('''{% extends 'layout.html' %}
{% block content %}
<section>
  <h2>Contact Us</h2>
  <form method="post" action="{{ url_for('contact') }}">
    <label for="name">Name</label>
    <input id="name" name="name" required maxlength="100">

    <label for="email">Email</label>
    <input id="email" name="email" type="email" required maxlength="120">

    <label for="message">Message</label>
    <textarea id="message" name="message" required maxlength="2000"></textarea>

    <button type="submit">Send</button>
  </form>
</section>
{% endblock %}''')

# Add a minimal stylesheet if missing
STYLE_CSS = STATIC_DIR / 'style.css'
if not STYLE_CSS.exists():
    STYLE_CSS.write_text('''*{box-sizing:border-box}body{font-family:Inter,Arial,Helvetica,sans-serif;margin:0;background:#fafafa;color:#111}a{color:inherit}header.site-header{background:#003366;color:#fff;padding:20px 0}header.site-header .container{display:flex;justify-content:space-between;align-items:center;max-width:1100px;margin:0 auto;padding:0 16px}header nav a{margin-left:12px;color:#dfefff;text-decoration:none}.container{max-width:1100px;margin:24px auto;padding:0 16px}.grid{display:grid;grid-template-columns:repeat(auto-fit,minmax(220px,1fr));gap:12px}figure{background:#fff;padding:8px;border-radius:6px}img{width:100%;height:180px;object-fit:cover;border-radius:4px}.site-footer{background:#111;color:#ddd;padding:16px;text-align:center;margin-top:40px}.flashes{list-style:none;padding:0}form{display:grid;gap:8px;max-width:640px}input,textarea{padding:8px;border:1px solid #ccc;border-radius:4px}button{padding:10px 14px;border:none;border-radius:6px;background:#0077cc;color:#fff}''')

# --- Integrated Morocco tourism data (merged into app for direct rendering) ---
MOROCCO_DATA = {
    'cities': [
        {
            'name': 'Marrakech',
            'desc': 'Known as the Red City, Marrakech is famous for Jemaa el-Fnaa, Koutoubia Mosque, Majorelle Garden, palaces, souks, and traditional riads.',
            'images': ['marrakech1.jpg','marrakech2.jpg','marrakech3.jpg']
        },
        {
            'name': 'Fes',
            'desc': 'The spiritual capital of Morocco. Home to Al-Qarawiyyin University, narrow medieval streets, leather tanneries, and ancient madrasas.',
            'images': ['fes1.jpg','fes2.jpg']
        },
        {
            'name': 'Chefchaouen',
            'desc': 'The famous Blue City of Morocco located in the Rif Mountains, known for its blue-washed houses and relaxed vibe.',
            'images': ['chefchaouen1.jpg','chefchaouen2.jpg']
        },
        {
            'name': 'Agadir',
            'desc': 'A coastal city known for its beaches, resorts, and sunny weather year-round.',
            'images': ['agadir1.jpg']
        },
        {
            'name': 'Tangier',
            'desc': 'A port city connecting Africa and Europe, rich with history and modern culture.',
            'images': ['tangier1.jpg']
        },
        {
            'name': 'Sahara Desert',
            'desc': 'Gold sand dunes, camel treks, starry skies, and Berber camps in Merzouga and Zagora.',
            'images': ['sahara1.jpg','sahara2.jpg']
        }
    ]
}

# Route to display all cities with details
@app.route('/cities')
def cities():
    return render_template('cities.html', data=MOROCCO_DATA, title='Explore Morocco Cities') (cities, attractions, descriptions) ---
MOROCCO_DATA = {
    'cities': [
        {
            'name': 'Marrakech',
            'desc': 'Known as the Red City, Marrakech is famous for Jemaa el-Fnaa, Koutoubia Mosque, Majorelle Garden, palaces, souks, and traditional riads.',
            'images': ['marrakech1.jpg','marrakech2.jpg','marrakech3.jpg']
        },
        {
            'name': 'Fes',
            'desc': 'The spiritual capital of Morocco. Home to Al-Qarawiyyin University, narrow medieval streets, leather tanneries, and ancient madrasas.',
            'images': ['fes1.jpg','fes2.jpg']
        },
        {
            'name': 'Chefchaouen',
            'desc': 'The famous Blue City of Morocco located in the Rif Mountains, known for its blue-washed houses and relaxed vibe.',
            'images': ['chefchaouen1.jpg','chefchaouen2.jpg']
        },
        {
            'name': 'Agadir',
            'desc': 'A coastal city known for its beaches, resorts, and sunny weather year-round.',
            'images': ['agadir1.jpg']
        },
        {
            'name': 'Tangier',
            'desc': 'A port city connecting Africa and Europe, rich with history and modern culture.',
            'images': ['tangier1.jpg']
        },
        {
            'name': 'Sahara Desert',
            'desc': 'Gold sand dunes, camel treks, starry skies, and Berber camps in Merzouga and Zagora.',
            'images': ['sahara1.jpg','sahara2.jpg']
        },
    ]
}

# Flask app
app = Flask(__name__, static_folder=str(STATIC_DIR), template_folder=str(TEMPLATES_DIR))
app.config['SECRET_KEY'] = os.getenv('SECRET_KEY', 'change-this-secret-in-production')
app.config['SESSION_COOKIE_SECURE'] = True
app.config['SESSION_COOKIE_HTTPONLY'] = True
app.config['SESSION_COOKIE_SAMESITE'] = 'Lax'

# Content Security Policy: allow self resources and images from self
csp = {
    'default-src': "'self'",
    'img-src': "'self' data:",
    'script-src': "'self' 'unsafe-inline'",
    'style-src': "'self' 'unsafe-inline'",
}

# Talisman adds security headers (HSTS, CSP, Referrer-Policy, etc.)
Talisman(app, content_security_policy=csp, force_https=not app.debug)

# Rate limiter to protect endpoints (e.g., contact form)
limiter = Limiter(app, key_func=get_remote_address, default_limits=["200 per day", "50 per hour"])

# Helper to list images (only filenames, no user input used in path building)
def list_gallery_images():
    # only include common image extensions
    allowed_ext = {'.jpg', '.jpeg', '.png', '.webp', '.gif'}
    files = []
    for p in IMAGES_DIR.iterdir():
        if p.is_file() and p.suffix.lower() in allowed_ext:
            files.append(p.name)
    # sort by name
    files.sort()
    return files

@app.context_processor
def inject_year():
    return {'year': 2025}

@app.route('/')
def index():
    return render_template('index.html', title='Home - Visit Morocco')

@app.route('/about')
def about():
    return render_template('about.html', title='About - Visit Morocco')

@app.route('/gallery')
def gallery():
    images = list_gallery_images()
    return render_template('gallery.html', images=images, title='Gallery - Visit Morocco')

@app.route('/contact', methods=['GET', 'POST'])
@limiter.limit("10 per hour")
def contact():
    if request.method == 'POST':
        # Basic input handling: escape user data and validate lengths
        name = escape(request.form.get('name', ''))[:100]
        email = escape(request.form.get('email', ''))[:120]
        message = escape(request.form.get('message', ''))[:2000]

        # Very simple validation
        if not name or not email or not message:
            flash('Please fill all required fields.')
            return redirect(url_for('contact'))

        # In a real app: store safely (parameterized DB), send email via secure mail service, validate email format server-side
        # Here we just simulate success
        flash('Thank you — your message was received. We will reply to ' + email)
        return redirect(url_for('index'))

    return render_template('contact.html', title='Contact - Visit Morocco')

# Small healthcheck route (avoid exposing internal details)
@app.route('/health')
def health():
    return ('ok', 200)

if __name__ == '__main__':
    # Development server: do NOT use in production. For production, run under gunicorn + nginx with TLS.
    debug_mode = os.getenv('FLASK_DEBUG', '0') == '1'
    # When not in debug, Talisman will enforce HTTPS. For local testing you may set FLASK_DEBUG=1
    app.run(debug=debug_mode, host='0.0.0.0', port=int(os.getenv('PORT', 5000)))