<template>
  <div class="home-page">
    <!-- Navigation Bar -->
    <nav class="navbar">
      <div class="container">
        <div class="logo">
          <img class="logo-image" src="@/assets/FullLogo.png" alt="2GN" />
          <h1>2GN Construction</h1>
        </div>
        <ul class="nav-links">
          <li><a href="#home">Home</a></li>
          <li><a href="#about">About</a></li>
          <li><a href="#services">Services</a></li>
          <li><a href="#projects">Projects</a></li>
          <li><a href="#projects">Gallery</a></li>
          <li><a href="#contact">Contact</a></li>
        </ul>
        <button class="menu-toggle" @click="toggleMenu">☰</button>
      </div>
    </nav>

    <!-- Hero Section -->
    <section id="home" class="hero">
      <div class="hero-overlay">
        <div class="hero-content">
          <h1 class="hero-title">Building Your Dreams Into Reality</h1>
          <p class="hero-subtitle">Professional Construction Services Since 2021</p>
          <div class="hero-buttons">
            <a class="btn btn-primary" href="#contact">Get A Quote</a>
            <button class="btn btn-secondary">Our Projects</button>
          </div>
        </div>
      </div>
    </section>

    <!-- About Section -->
    <section id="about" class="about">
      <div class="container">
        <div class="section-header">
          <h2>About Us</h2>
          <div class="underline"></div>
        </div>
        <div class="about-content">
          <div class="about-text">
            <h3>15+ Years of Excellence in Construction</h3>
            <p>
              2GN Construction is a leading construction company specializing in residential, 
              commercial. With over 40 years of experience, we have 
              built a reputation for delivering high-quality projects on time and within budget.
            </p>
            <p>
              Our team of experienced professionals is dedicated to bringing your vision to life 
              with precision, innovation, and commitment to excellence.
            </p>
            <div class="stats">
              <div class="stat-item">
                <h4>500+</h4>
                <p>Projects Completed</p>
              </div>
              <div class="stat-item">
                <h4>98%</h4>
                <p>Client Satisfaction</p>
              </div>
              <div class="stat-item">
                <h4>50+</h4>
                <p>Expert Team Members</p>
              </div>
            </div>
          </div>
          <div class="about-image">
            <div class="image-placeholder">
              <span>🏗️</span>
            </div>
          </div>
        </div>
      </div>
    </section>

    <!-- Services Section -->
    <section id="services" class="services">
      <div class="container">
        <div class="section-header">
          <h2>Our Services</h2>
          <div class="underline"></div>
          <p>Comprehensive construction solutions for all your needs</p>
        </div>
        <div class="services-grid">
          <div class="service-card" v-for="service in services" :key="service.id">
            <div class="service-icon">{{ service.icon }}</div>
            <h3>{{ service.title }}</h3>
            <p>{{ service.description }}</p>
          </div>
        </div>
      </div>
    </section>

    <!-- Projects Section -->
    <section id="projects" class="projects">
      <div class="container">
        <div class="section-header">
          <h2>Project Gallery</h2>
          <div class="underline"></div>
          <p>Name, description, and photos from our database</p>
        </div>
        <div v-if="projectsLoading" class="gallery-status">Loading projects...</div>
        <div v-else-if="projectCards.length === 0" class="gallery-status">No projects yet.</div>
        <div v-else class="projects-grid">
          <div class="project-card" v-for="project in projectCards" :key="project.projectID" @click="openProjectGallery(project)">
            <div class="project-image">
              <img v-if="project.imageID" :src="imageUrl(project.imageID)" :alt="project.project_name" />
              <div v-else class="project-placeholder">🏗️</div>
            </div>
            <div class="project-info">
              <h3>{{ project.project_name }}</h3>
              <p>{{ project.project_description || 'Description coming soon.' }}</p>
            </div>
          </div>
        </div>
      </div>
    </section>

    <div v-if="isGalleryOpen" class="gallery-modal-overlay" @click.self="closeProjectGallery">
      <div class="gallery-modal">
        <button class="gallery-close" @click="closeProjectGallery" aria-label="Close gallery">×</button>
        <div class="gallery-modal-header">
          <h3>{{ selectedProject.project_name }}</h3>
          <p>{{ selectedProject.project_description || 'Description coming soon.' }}</p>
        </div>
        <div v-if="selectedProjectImages.length > 0" class="gallery-modal-main">
          <button
            v-if="selectedProjectImages.length > 1"
            class="gallery-nav gallery-prev"
            @click="prevGalleryImage"
            aria-label="Previous image"
          >
            ‹
          </button>
          <img
            :src="imageUrl(selectedProjectImages[activeGalleryIndex].imageID)"
            :alt="selectedProject.project_name"
          />
          <button
            v-if="selectedProjectImages.length > 1"
            class="gallery-nav gallery-next"
            @click="nextGalleryImage"
            aria-label="Next image"
          >
            ›
          </button>
        </div>
        <div v-else class="gallery-modal-empty">No images available for this project yet.</div>
        <div v-if="selectedProjectImages.length > 1" class="gallery-thumbnails">
          <button
            v-for="(image, index) in selectedProjectImages"
            :key="image.imageID"
            class="gallery-thumb"
            :class="{ active: index === activeGalleryIndex }"
            @click="setGalleryImage(index)"
            :aria-label="`View image ${index + 1}`"
          >
            <img :src="imageUrl(image.imageID)" :alt="`${selectedProject.project_name} ${index + 1}`" />
          </button>
        </div>
      </div>
    </div>

    <!-- Contact + Quote Section -->
    <section id="contact" class="contact">
      <div class="container">
        <div class="section-header">
          <h2>Contact & Get a Quote</h2>
          <div class="underline"></div>
          <p>Tell us about your project and we’ll prepare a fast, accurate estimate.</p>
        </div>
        <div class="contact-content">
          <div class="contact-info">
            <div class="info-item">
              <div class="info-icon">📍</div>
              <div>
                <h4>Address</h4>
                <p>6299 W Contadora Dr, West Valley City, Utah 84128</p>
              </div>
            </div>
            <div class="info-item">
              <div class="info-icon">📞</div>
              <div>
                <h4>Phone</h4>
                <p>+1 (801) 381-9195</p>
              </div>
            </div>
            <div class="info-item">
              <div class="info-icon">✉️</div>
              <div>
                <h4>Email</h4>
                <p>abel@2gnconstruction.com</p>
              </div>
            </div>
            <div class="info-item">
              <div class="info-icon">🕐</div>
              <div>
                <h4>Working Hours</h4>
                <p>Mon - Fri: 8:00 AM - 6:00 PM</p>
                <p>Sat: by appointment only</p>
                <p>Sun: Closed</p>
              </div>
            </div>
          </div>
          <div class="contact-form">
            <form @submit.prevent="handleSubmit">
              <div class="form-group">
                <input type="text" placeholder="Your Name" v-model="form.name" required>
              </div>
              <div class="form-group">
                <input type="email" placeholder="Your Email" v-model="form.email" required>
              </div>
              <div class="form-group">
                <input type="tel" placeholder="Your Phone" v-model="form.phone">
              </div>
              <div class="form-group">
                <select v-model="form.projectType" required>
                  <option disabled value="">Project Type</option>
                  <option>Residential</option>
                  <option>Commercial</option>
                  <option>Renovation</option>
                  <option>Other</option>
                </select>
              </div>
              <div class="form-group">
                <select v-model="form.budget" required>
                  <option disabled value="">Estimated Budget</option>
                  <option>Under $25k</option>
                  <option>$25k - $75k</option>
                  <option>$75k - $150k</option>
                  <option>$150k+</option>
                </select>
              </div>
              <div class="form-group">
                <input type="text" placeholder="Preferred Timeline (e.g., Q2 2026)" v-model="form.timeline">
              </div>
              <div class="form-group">
                <textarea placeholder="Project Details" v-model="form.message" rows="5" required></textarea>
              </div>
              <button type="submit" class="btn btn-primary" :disabled="submitting">
                {{ submitting ? 'Sending...' : 'Request Quote' }}
              </button>
              <p v-if="submitMessage" class="form-success">{{ submitMessage }}</p>
              <p v-if="submitError" class="form-error">{{ submitError }}</p>
            </form>
          </div>
        </div>
      </div>
    </section>

    <!-- Footer -->
    <footer class="footer">
      <div class="container">
        <div class="footer-content">
          <div class="footer-section">
            <h3>2GN Construction</h3>
            <p>Building excellence, one project at a time.</p>
          </div>
          <div class="footer-section">
            <h4>Quick Links</h4>
            <ul>
              <li><a href="#home">Home</a></li>
              <li><a href="#about">About</a></li>
              <li><a href="#services">Services</a></li>
              <li><a href="#projects">Projects</a></li>
            </ul>
          </div>
          <div class="footer-section">
            <h4>Services</h4>
            <ul>
              <li>Residential Construction</li>
              <li>Commercial Building</li>
              <li>Renovation & Remodeling</li>
              <li>Project Management</li>
            </ul>
          </div>
          <div class="footer-section">
            <h4>Follow Us</h4>
            <div class="social-links">
              <a href="#">Facebook</a>
              <a href="#">Instagram</a>
              <a href="#">LinkedIn</a>
            </div>
          </div>
        </div>
        <div class="footer-bottom">
          <p>
            &copy; 2026 2GN Construction. All rights reserved.
            <a class="footer-cms" href="#/admin">ADMIN</a>
          </p>
        </div>
      </div>
    </footer>
  </div>
</template>

<script>
const API_BASE = process.env.VUE_APP_API_BASE || 'http://localhost:3000';

export default {
  name: 'HomePage',
  data() {
    return {
      menuOpen: false,
      projectsFromDb: [],
      projectImages: [],
      projectsLoading: false,
      isGalleryOpen: false,
      selectedProject: null,
      activeGalleryIndex: 0,
      previousBodyOverflow: '',
      services: [
        {
          id: 1,
          icon: '🏘️',
          title: 'Residential Construction',
          description: 'Custom homes and residential developments built to your specifications with quality craftsmanship.'
        },
        {
          id: 2,
          icon: '🏢',
          title: 'Commercial Building',
          description: 'Modern commercial structures designed for functionality, aesthetics, and long-term value.'
        },
        {
          id: 3,
          icon: '🏭',
          title: 'Industrial Projects',
          description: 'Large-scale industrial construction projects with focus on efficiency and safety standards.'
        },
        {
          id: 4,
          icon: '🔨',
          title: 'Renovation & Remodeling',
          description: 'Transform existing spaces with our expert renovation and remodeling services.'
        },
        {
          id: 5,
          icon: '📋',
          title: 'Project Management',
          description: 'Comprehensive project management ensuring timely delivery and budget compliance.'
        },
        {
          id: 6,
          icon: '🎨',
          title: 'Design & Planning',
          description: 'Professional architectural design and planning services from concept to completion.'
        }
      ],
      form: {
        name: '',
        email: '',
        phone: '',
        projectType: '',
        budget: '',
        timeline: '',
        message: ''
      },
      submitting: false,
      submitMessage: '',
      submitError: ''
    }
  },
  mounted() {
    this.fetchProjectCards();
    document.addEventListener('keydown', this.handleGalleryKeydown);
  },
  beforeUnmount() {
    this.unlockBodyScroll();
    document.removeEventListener('keydown', this.handleGalleryKeydown);
  },
  computed: {
    projectCards() {
      return this.projectsFromDb.map((project) => {
        const image = this.projectImages.find((item) => item.projectID === project.projectID);
        return {
          ...project,
          imageID: image ? image.imageID : null
        };
      });
    },
    selectedProjectImages() {
      if (!this.selectedProject) return [];
      return this.projectImages.filter((image) => image.projectID === this.selectedProject.projectID);
    }
  },
  methods: {
    toggleMenu() {
      this.menuOpen = !this.menuOpen;
    },
    async fetchProjectCards() {
      this.projectsLoading = true;
      try {
        const [projectResponse, imageResponse] = await Promise.all([
          fetch(`${API_BASE}/project`),
          fetch(`${API_BASE}/project-image`)
        ]);
        this.projectsFromDb = await projectResponse.json();
        this.projectImages = await imageResponse.json();
      } catch (error) {
        console.error('Failed to load projects', error);
        this.projectsFromDb = [];
        this.projectImages = [];
      } finally {
        this.projectsLoading = false;
      }
    },
    imageUrl(id) {
      return `${API_BASE}/project-image/${id}/file`;
    },
    openProjectGallery(project) {
      this.selectedProject = project;
      this.activeGalleryIndex = 0;
      this.isGalleryOpen = true;
      this.lockBodyScroll();
    },
    closeProjectGallery() {
      this.isGalleryOpen = false;
      this.selectedProject = null;
      this.activeGalleryIndex = 0;
      this.unlockBodyScroll();
    },
    lockBodyScroll() {
      this.previousBodyOverflow = document.body.style.overflow;
      document.body.style.overflow = 'hidden';
    },
    unlockBodyScroll() {
      document.body.style.overflow = this.previousBodyOverflow;
    },
    prevGalleryImage() {
      if (this.selectedProjectImages.length < 2) return;
      const nextIndex = this.activeGalleryIndex - 1;
      this.activeGalleryIndex = nextIndex < 0 ? this.selectedProjectImages.length - 1 : nextIndex;
    },
    nextGalleryImage() {
      if (this.selectedProjectImages.length < 2) return;
      this.activeGalleryIndex = (this.activeGalleryIndex + 1) % this.selectedProjectImages.length;
    },
    setGalleryImage(index) {
      this.activeGalleryIndex = index;
    },
    handleGalleryKeydown(event) {
      if (!this.isGalleryOpen) return;
      if (event.key === 'Escape') {
        this.closeProjectGallery();
      }
      if (event.key === 'ArrowLeft') {
        this.prevGalleryImage();
      }
      if (event.key === 'ArrowRight') {
        this.nextGalleryImage();
      }
    },
    async handleSubmit() {
      this.submitting = true;
      this.submitMessage = '';
      this.submitError = '';
      try {
        const response = await fetch(`${API_BASE}/quote`, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify(this.form)
        });

        if (!response.ok) {
          const payload = await response.json().catch(() => null);
          this.submitError = payload?.message || 'Failed to send request.';
          return;
        }

        const payload = await response.json().catch(() => null);
        this.submitMessage = payload?.message || 'Thanks for your request! We will contact you shortly.';
        this.form = {
          name: '',
          email: '',
          phone: '',
          projectType: '',
          budget: '',
          timeline: '',
          message: ''
        };
      } catch (error) {
        console.error('Quote request failed', error);
        this.submitError = 'Failed to send request.';
      } finally {
        this.submitting = false;
      }
    }
  }
}
</script>

<style scoped>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

.home-page {
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  color: #333;
}

.container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 0 20px;
}

/* Navigation */
.navbar {
  background: #1a1a1a;
  color: white;
  padding: 1rem 0;
  position: sticky;
  top: 0;
  z-index: 100;
  box-shadow: 0 2px 10px rgba(0,0,0,0.1);
}

.navbar .container {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.logo h1 {
  font-size: 1.8rem;
  color: #ff6b35;
}

.logo {
  display: flex;
  align-items: center;
  gap: 0.75rem;
}

.logo-image {
  height: 44px;
  width: auto;
  display: block;
}

.nav-links {
  display: flex;
  list-style: none;
  gap: 2rem;
}

.nav-links a {
  color: white;
  text-decoration: none;
  font-weight: 500;
  transition: color 0.3s;
}

.nav-links a:hover {
  color: #ff6b35;
}

.menu-toggle {
  display: none;
  background: none;
  border: none;
  color: white;
  font-size: 1.5rem;
  cursor: pointer;
}

/* Hero Section */
.hero {
  height: 600px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  position: relative;
  display: flex;
  align-items: center;
  justify-content: center;
}

.hero-overlay {
  background: rgba(0, 0, 0, 0.4);
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
}

.hero-content {
  text-align: center;
  color: white;
  padding: 2rem;
}

.hero-title {
  font-size: 3.5rem;
  margin-bottom: 1rem;
  font-weight: bold;
  text-shadow: 2px 2px 4px rgba(0,0,0,0.3);
}

.hero-subtitle {
  font-size: 1.5rem;
  margin-bottom: 2rem;
  opacity: 0.95;
}

.hero-buttons {
  display: flex;
  gap: 1rem;
  justify-content: center;
}

.btn {
  padding: 1rem 2rem;
  font-size: 1rem;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  font-weight: 600;
  transition: all 0.3s;
}

.btn-primary {
  background: #ff6b35;
  color: white;
}

.btn-primary:hover {
  background: #e55a2b;
  transform: translateY(-2px);
  box-shadow: 0 5px 15px rgba(255,107,53,0.4);
}

.btn-secondary {
  background: transparent;
  color: white;
  border: 2px solid white;
}

.btn-secondary:hover {
  background: white;
  color: #667eea;
}

/* Section Headers */
.section-header {
  text-align: center;
  margin-bottom: 3rem;
}

.section-header h2 {
  font-size: 2.5rem;
  margin-bottom: 0.5rem;
  color: #1a1a1a;
}

.underline {
  width: 80px;
  height: 4px;
  background: #ff6b35;
  margin: 0 auto 1rem;
}

.section-header p {
  font-size: 1.1rem;
  color: #666;
}

/* About Section */
.about {
  padding: 5rem 0;
  background: #f8f9fa;
}

.about-content {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 3rem;
  align-items: center;
}

.about-text h3 {
  font-size: 2rem;
  margin-bottom: 1rem;
  color: #1a1a1a;
}

.about-text p {
  font-size: 1.1rem;
  line-height: 1.8;
  color: #666;
  margin-bottom: 1rem;
}

.stats {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 2rem;
  margin-top: 2rem;
}

.stat-item {
  text-align: center;
}

.stat-item h4 {
  font-size: 2.5rem;
  color: #ff6b35;
  margin-bottom: 0.5rem;
}

.stat-item p {
  font-size: 1rem;
  color: #666;
}

.about-image {
  display: flex;
  justify-content: center;
  align-items: center;
}

.image-placeholder {
  width: 400px;
  height: 400px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  border-radius: 10px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 8rem;
  box-shadow: 0 10px 30px rgba(0,0,0,0.2);
}

/* Services Section */
.services {
  padding: 5rem 0;
}

.services-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: 2rem;
}

.service-card {
  background: white;
  padding: 2rem;
  border-radius: 10px;
  box-shadow: 0 5px 15px rgba(0,0,0,0.1);
  text-align: center;
  transition: transform 0.3s;
}

.service-card:hover {
  transform: translateY(-10px);
  box-shadow: 0 10px 25px rgba(0,0,0,0.15);
}

.service-icon {
  font-size: 3rem;
  margin-bottom: 1rem;
}

.service-card h3 {
  font-size: 1.5rem;
  margin-bottom: 1rem;
  color: #1a1a1a;
}

.service-card p {
  color: #666;
  line-height: 1.6;
}

/* Projects Section */
.projects {
  padding: 5rem 0;
  background: #f8f9fa;
}

.projects-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: 2rem;
}

.project-card {
  background: white;
  border-radius: 10px;
  overflow: hidden;
  box-shadow: 0 5px 15px rgba(0,0,0,0.1);
  transition: transform 0.3s;
  cursor: pointer;
}

.project-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 10px 25px rgba(0,0,0,0.15);
}

.project-image {
  width: 100%;
  height: 200px;
  overflow: hidden;
}

.project-placeholder {
  width: 100%;
  height: 100%;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 4rem;
}

.project-image img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.project-info {
  padding: 1.5rem;
}

.project-info h3 {
  font-size: 1.3rem;
  margin-bottom: 0.5rem;
  color: #1a1a1a;
}

.project-category {
  color: #ff6b35;
  font-weight: 600;
  margin-bottom: 0.5rem;
}

.project-info p {
  color: #666;
  line-height: 1.6;
}

.gallery-modal-overlay {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.7);
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 1rem;
  z-index: 1000;
}

.gallery-modal {
  width: 100%;
  max-width: 900px;
  max-height: 90vh;
  overflow-y: auto;
  background: #fff;
  border-radius: 12px;
  padding: 1.5rem;
  position: relative;
}

.gallery-close {
  position: absolute;
  top: 0.5rem;
  right: 0.75rem;
  border: none;
  background: transparent;
  font-size: 2rem;
  line-height: 1;
  cursor: pointer;
  color: #374151;
}

.gallery-modal-header {
  margin-bottom: 1rem;
  padding-right: 2rem;
}

.gallery-modal-header h3 {
  font-size: 1.5rem;
  color: #1a1a1a;
  margin-bottom: 0.5rem;
}

.gallery-modal-header p {
  color: #6b7280;
}

.gallery-modal-main {
  position: relative;
  border-radius: 10px;
  overflow: hidden;
  background: #f3f4f6;
}

.gallery-modal-main img {
  width: 100%;
  max-height: 60vh;
  object-fit: cover;
  display: block;
}

.gallery-nav {
  position: absolute;
  top: 50%;
  transform: translateY(-50%);
  width: 40px;
  height: 40px;
  border: none;
  border-radius: 999px;
  background: rgba(0, 0, 0, 0.55);
  color: #fff;
  font-size: 1.5rem;
  cursor: pointer;
}

.gallery-prev {
  left: 0.75rem;
}

.gallery-next {
  right: 0.75rem;
}

.gallery-thumbnails {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(90px, 1fr));
  gap: 0.75rem;
  margin-top: 1rem;
}

.gallery-thumb {
  border: 2px solid transparent;
  border-radius: 8px;
  padding: 0;
  overflow: hidden;
  cursor: pointer;
  background: #fff;
}

.gallery-thumb img {
  width: 100%;
  height: 70px;
  object-fit: cover;
  display: block;
}

.gallery-thumb.active {
  border-color: #ff6b35;
}

.gallery-modal-empty {
  text-align: center;
  color: #6b7280;
  padding: 2rem 1rem;
  background: #f9fafb;
  border-radius: 10px;
}

/* Gallery Section */
.gallery {
  padding: 5rem 0;
  background: #ffffff;
}

.gallery-status {
  text-align: center;
  color: #6b7280;
  margin-top: 1.5rem;
}

.gallery-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(220px, 1fr));
  gap: 1.5rem;
  margin-top: 2rem;
}

.gallery-card {
  border-radius: 12px;
  overflow: hidden;
  box-shadow: 0 8px 20px rgba(0,0,0,0.08);
  background: #f4f4f4;
  height: 200px;
}

.gallery-card img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

/* Contact Section */
.contact {
  padding: 5rem 0;
}

.contact-content {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 3rem;
}

.contact-info {
  display: flex;
  flex-direction: column;
  gap: 2rem;
}

.info-item {
  display: flex;
  gap: 1rem;
}

.info-icon {
  font-size: 2rem;
}

.info-item h4 {
  font-size: 1.2rem;
  margin-bottom: 0.5rem;
  color: #1a1a1a;
}

.info-item p {
  color: #666;
  line-height: 1.6;
}

.contact-form {
  background: #f8f9fa;
  padding: 2rem;
  border-radius: 10px;
}

.form-group {
  margin-bottom: 1.5rem;
}

.form-group input,
.form-group textarea,
.form-group select {
  width: 100%;
  padding: 1rem;
  border: 1px solid #ddd;
  border-radius: 5px;
  font-size: 1rem;
  font-family: inherit;
  background: #fff;
}

.form-group input:focus,
.form-group textarea:focus,
.form-group select:focus {
  outline: none;
  border-color: #667eea;
}

.form-group textarea {
  resize: vertical;
}

.form-success {
  margin-top: 1rem;
  color: #16a34a;
  font-weight: 600;
}

.form-error {
  margin-top: 1rem;
  color: #dc2626;
  font-weight: 600;
}

.btn:disabled {
  opacity: 0.7;
  cursor: default;
}

/* Footer */
.footer {
  background: #1a1a1a;
  color: white;
  padding: 3rem 0 1rem;
}

.footer-content {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 2rem;
  margin-bottom: 2rem;
}

.footer-section h3 {
  font-size: 1.5rem;
  margin-bottom: 1rem;
  color: #ff6b35;
}

.footer-section h4 {
  font-size: 1.2rem;
  margin-bottom: 1rem;
}

.footer-section p {
  color: #ccc;
  line-height: 1.6;
}

.footer-section ul {
  list-style: none;
}

.footer-section li {
  margin-bottom: 0.5rem;
  color: #ccc;
}

.footer-section a {
  color: #ccc;
  text-decoration: none;
  transition: color 0.3s;
}

.footer-section a:hover {
  color: #ff6b35;
}

.social-links {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.footer-bottom {
  text-align: center;
  padding-top: 2rem;
  border-top: 1px solid #333;
  color: #999;
}

.footer-cms {
  margin-left: 12px;
  font-size: 0.85rem;
  color: #999;
  opacity: 0.6;
  text-decoration: none;
}

.footer-cms:hover {
  opacity: 1;
  color: #ccc;
}

/* Responsive Design */
@media (max-width: 768px) {
  .nav-links {
    display: none;
  }
  
  .menu-toggle {
    display: block;
  }
  
  .hero-title {
    font-size: 2rem;
  }
  
  .hero-subtitle {
    font-size: 1.1rem;
  }
  
  .hero-buttons {
    flex-direction: column;
    align-items: center;
  }
  
  .about-content,
  .contact-content {
    grid-template-columns: 1fr;
  }
  
  .stats {
    grid-template-columns: 1fr;
  }
  
  .image-placeholder {
    width: 300px;
    height: 300px;
    font-size: 6rem;
  }
  
  .section-header h2 {
    font-size: 2rem;
  }

  .gallery-modal {
    padding: 1rem;
  }

  .gallery-modal-main img {
    max-height: 45vh;
  }
}

@media (max-width: 480px) {
  .hero-title {
    font-size: 1.5rem;
  }
  
  .btn {
    padding: 0.8rem 1.5rem;
    font-size: 0.9rem;
  }
}
</style>
