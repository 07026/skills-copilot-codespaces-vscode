<!DOCTYPE html>
<html lang="en" dir="ltr">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Morocco Adventures | Book Your Dream Tour</title>
    <link href="https:                                                                                               
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css">
    <style>
      /* --- Base CSS & Morocco-Inspired Palette --- */
      :root {
        --primary-color: #008080; /* Teal/Moroccan Green */
        --accent-color: #B8860B; /* Gold/Brass accent */
        --secondary-color: #3d3d3d;
        --light-bg: #f7f7f7;
        --white: #fff;
      }
      * {
        margin: 0;
        padding: 0;
        box-sizing: border-box;
        font-family: 'Poppins', sans-serif;
      }
      body {
        line-height: 1.6;
        background-color: var(--light-bg);
        color: var(--secondary-color);
      }
      a {
        text-decoration: none;
        color: inherit;
      }
      ul {
        list-style: none;
      }
      /* --- Global Styles --- */
      .container {
        width: 90%;
        max-width: 1200px;
        margin: auto;
      }
      .section-title {
        text-align: center;
        margin-bottom: 4rem;
        color: var(--secondary-color);
        font-size: 2.5rem;
        font-weight: 700;
      }
      .btn {
        display: inline-block;
        background: var(--primary-color);
        color: var(--white);
        padding: 15px 35px;
        border-radius: 5px;
        font-weight: 600;
        transition: 0.3s;
        text-transform: uppercase;
        letter-spacing: 1px;
        border: none;
        cursor: pointer;
      }
      .btn:hover {
        background: var(--accent-color);
      }
      /* --- Navbar (Same as before) --- */
      header {
        background: var(--white);
        box-shadow: 0 2px 8px rgba(0,0,0,0.1);
        position: fixed;
        width: 100%;
        top: 0;
        z-index: 1000;
      }
      nav {
        display: flex;
        justify-content: space-between;
        align-items: center;
        padding: 1rem 0;
      }
      .logo {
        font-size: 1.5rem;
        font-weight: 700;
        color: var(--primary-color);
      }
      nav ul {
        display: flex;
      }
      nav ul li {
        margin-left: 25px;
      }
      nav ul li a {
        font-weight: 500;
        transition: 0.3s;
      }
      nav ul li a:hover {
        color: var(--accent-color);
      }
      /* --- Hero Section (Same as before) --- */
      .hero {
        height: 100vh;
        background: linear-gradient(rgba(0,0,0,0.6), rgba(0,0,0,0.4)), url('https://images.unsplash.com/photo-1549791404-3b2d1c6762fe?ixlib=rb-1.2.1&auto=format&fit=crop&w=1350&q=80');
        background-size: cover;
        background-position: center;
        display: flex;
        align-items: center;
        justify-content: center;
        text-align: center;
        color: var(--white);
        margin-top: 60px;
      }
      .hero h1 {
        font-size: 4rem;
        margin-bottom: 1rem;
        font-weight: 700;
      }
      .hero p {
        font-size: 1.3rem;
        margin-bottom: 2.5rem;
        max-width: 700px;
        margin-left: auto;
        margin-right: auto;
      }
      /* --- Destinations Section (Same as before) --- */
      .destinations {
        padding: 5rem 0 3rem;
      }
      .grid-3 {
        display: grid;
        grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
        gap: 30px;
      }
      .card {
        background: var(--white);
        border-radius: 10px;
        overflow: hidden;
        box-shadow: 0 8px 20px rgba(0,0,0,0.1);
        transition: transform 0.4s ease, box-shadow 0.4s ease;
      }
      .card:hover {
        transform: translateY(-10px);
        box-shadow: 0 15px 30px rgba(0,0,0,0.2);
      }
      .card img {
        width: 100%;
        height: 250px;
        object-fit: cover;
      }
      .card-content {
        padding: 1.5rem;
      }
      .card-content h3 {
        margin-bottom: 10px;
        color: var(--primary-color);
        font-weight: 600;
      }
      .card-content p {
        font-size: 0.95rem;
        color: #666;
        margin-bottom: 15px;
      }
      .card-content .btn {
        padding: 10px 20px;
        font-size: 0.85rem;
      }
      /* --- NEW: Testimonials Section --- */
      .testimonials {
        background-color: var(--primary-color);
        color: var(--white);
        padding: 5rem 0;
      }
      .testimonial-title {
        color: var(--white);
      }
      .testimonial-grid {
        display: flex;
        gap: 30px;
        overflow-x: auto; /* Enable scrolling on smaller screens */
        padding-bottom: 20px;
      }
      .testimonial-card {
        background: rgba(255, 255, 255, 0.1);
        border-left: 5px solid var(--accent-color);
        padding: 25px;
        border-radius: 8px;
        min-width: 300px;
        flex: 1; /* Ensures cards are flexible */
      }
      .testimonial-card i {
        color: var(--accent-color);
        margin-bottom: 10px;
      }
      .testimonial-card p {
        font-style: italic;
        margin-bottom: 15px;
        font-size: 0.95rem;
      }
      .client-name {
        font-weight: 600;
        text-align: right;
        border-top: 1px solid rgba(255, 255, 255, 0.2);
        padding-top: 10px;
      }
      /* --- NEW: Booking Form Section --- */
      .booking-section {
        padding: 5rem 0;
        background-color: var(--white);
      }
      .booking-form {
        max-width: 600px;
        margin: auto;
        background: var(--light-bg);
        padding: 40px;
        border-radius: 10px;
        box-shadow: 0 10px 25px rgba(0,0,0,0.05);
      }
      .form-group {
        margin-bottom: 20px;
      }
      .form-group label {
        display: block;
        margin-bottom: 8px;
        font-weight: 600;
      }
      .form-group input, .form-group select, .form-group textarea {
        width: 100%;
        padding: 12px;
        border: 1px solid #ddd;
        border-radius: 5px;
        font-size: 1rem;
        transition: border-color 0.3s;
      }
      .form-group input:focus, .form-group select:focus, .form-group textarea:focus {
        border-color: var(--primary-color);
        outline: none;
      }
    </style>
  </head>
  <body>
    <header>
      <div class="container">
        <nav>
          <div class="logo">
            <i class="fas fa-mountain"></i> Morocco Adventures
          </div>
          <ul>
            <li><a href="                
            <li><a href="#destinations">Tours</a></li>
            <li><a href="                               
            <li><a href="#booking">Book Now</a></li>
            <li><a href="                          
          </ul>
        </nav>
      </div>
    </header>
    <section class="hero">
      <div class="container">
        <h1>Experience the Magic of Morocco</h1>
        <p>From the bustling souks of Marrakech to the silent, golden dunes of the Sahara, your adventure starts here.</p>
        <a href="#booking" class="btn">Book Your Tour</a>
      </div>
    </section>
    <section id="destinations" class="destinations container">
      <h2 class="section-title">Must-Visit Moroccan Destinations</h2>
      <
