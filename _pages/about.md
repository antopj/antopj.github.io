---
permalink: /about/
title: "About Me"
layout: page
---

<div class="about-container">
  <aside class="sidebar">
    <div class="navigation">
      <a href="/"><i class="fas fa-home"></i> Main Page</a>
      {% if site.categories.size > 0 %}
      <a href="/categories/"><i class="fas fa-folder-open"></i> Categories</a>
      {% endif %}
      {% if site.tags.size > 0 %}
      <a href="/tags/"><i class="fas fa-tags"></i> Tags</a>
      {% endif %}
    </div>
  </aside>

  <main class="main-content">
    <div class="profile">
      <img src="/assets/images/bio-photo.jpg" alt="Your Profile Picture">
      <h2>Coffee Loving Sys-Admin</h2>
      <p>Just sharing thoughts and info from the world of a Sys-Admin who runs on coffee. :)</p>
    </div>

    <div class="content-types">
      <div class="content-item">
        <i class="fas fa-graduation-cap fa-2x"></i>
        <h3>Tutorials</h3>
        <p>Step-by-step guides to help you learn and implement various tech.</p>
      </div>
      <div class="content-item">
        <i class="fas fa-book-open fa-2x"></i>
        <h3>Learning Paths</h3>
        <p>Roadmaps to navigate and master new technologies.</p>
      </div>
      <div class="content-item">
        <i class="fas fa-wrench fa-2x"></i>
        <h3>Troubleshooting</h3>
        <p>Solutions and tips for resolving common technical issues.</p>
      </div>
      <div class="content-item">
        <i class="fas fa-link fa-2x"></i>
        <h3>Useful Links</h3>
        <p>Collection of valuable resources and websites.</p>
      </div>
      <div class="content-item">
        <i class="fas fa-lightbulb fa-2x"></i>
        <h3>Recommended Tutorials</h3>
        <p>My personal picks for excellent learning materials.</p>
      </div>
    </div>
  </main>
</div>

<style>
.about-container {
  display: flex;
  font-family: sans-serif;
  color: #333;
  padding: 20px;
  gap: 20px; /* Space between sidebar and main content */
}

.sidebar {
  width: 200px; /* Adjust as needed */
  padding: 20px;
  /* Removed background color to blend better */
}

.navigation {
  display: flex;
  flex-direction: column;
  border-right: 1px solid #eee; /* Subtle separator */
  padding-right: 20px;
}

.navigation a {
  display: flex;
  align-items: center;
  padding: 10px 0;
  text-decoration: none;
  color: #007bff;
  font-weight: bold;
  margin-bottom: 5px;
}

.navigation a:hover {
  color: #0056b3;
}

.navigation a i {
  margin-right: 10px;
  width: 20px; /* Ensure consistent icon spacing */
  text-align: center;
}

.main-content {
  flex: 1;
  padding: 20px;
  background-color: #f9f9f9; /* Light background for main content */
  border-radius: 8px; /* Optional rounded corners */
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.05); /* Subtle shadow */
}

.profile {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-bottom: 30px;
}

.profile img {
  width: 100px;
  height: 100px;
  border-radius: 50%;
  object-fit: cover;
  margin-bottom: 10px;
}

.profile h2 {
  margin-top: 0;
  margin-bottom: 5px;
  font-size: 1.5em;
}

.content-types {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
  gap: 20px;
  margin-bottom: 30px;
  text-align: center;
}

.content-item {
  padding: 15px;
  border: 1px solid #eee;
  border-radius: 8px;
  background-color: white; /* White boxes for content items */
}

.content-item i {
  color: #007bff; /* Example accent color */
  margin-bottom: 10px;
}

.content-item h3 {
  margin-top: 0;
  font-size: 1.1em;
  margin-bottom: 5px;
}

.content-item p {
  font-size: 0.9em;
  color: #666;
}
</style>

<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css">
