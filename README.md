/* ═══════════════════════════════════════════════
   RIKI PHOTOS — main.js
   - Image protection (right-click, drag, keyboard)
   - Sticky nav scroll behaviour
   - Mobile menu toggle
   - Smooth scroll
   - Contact form basic handling
   © 2025 Riki Photos
═══════════════════════════════════════════════ */

(function () {
  'use strict';

  /* ── IMAGE PROTECTION ─────────────────────────── */

  // Block right-click on all photo areas
  document.addEventListener('contextmenu', function (e) {
    var protectedSelectors = [
      'img',
      '.protect-overlay',
      '.watermark-grid',
      '.hero-right',
      '.portfolio-item',
      '.portfolio-placeholder',
      '.travel-visual',
      '.about-visual',
      '.img-placeholder',
    ];
    var isProtected = protectedSelectors.some(function (sel) {
      return e.target.matches(sel) || e.target.closest(sel);
    });
    if (isProtected) {
      e.preventDefault();
      return false;
    }
  });

  // Block image drag
  document.addEventListener('dragstart', function (e) {
    if (e.target.tagName === 'IMG') {
      e.preventDefault();
      return false;
    }
  });

  // Block Ctrl/Cmd+S (Save Page) and Ctrl/Cmd+U (View Source)
  document.addEventListener('keydown', function (e) {
    var key = e.key.toLowerCase();
    if ((e.ctrlKey || e.metaKey) && (key === 's' || key === 'u')) {
      e.preventDefault();
      return false;
    }
  });

  // Block long-press selection on photos (mobile)
  var photoAreas = document.querySelectorAll(
    '.hero-right, .portfolio-item, .travel-visual, .about-visual, .img-placeholder'
  );
  photoAreas.forEach(function (el) {
    el.addEventListener('selectstart', function (e) { e.preventDefault(); });
    el.addEventListener('touchstart', function (e) {
      // prevent long-press context menu on mobile
      el.setAttribute('data-touch-start', Date.now());
    }, { passive: true });
  });

  /* ── STICKY NAV ───────────────────────────────── */

  var navbar = document.getElementById('navbar');
  if (navbar) {
    window.addEventListener('scroll', function () {
      if (window.scrollY > 20) {
        navbar.classList.add('scrolled');
      } else {
        navbar.classList.remove('scrolled');
      }
    }, { passive: true });
  }

  /* ── MOBILE MENU TOGGLE ───────────────────────── */

  var navToggle = document.querySelector('.nav-toggle');
  var navLinks  = document.querySelector('.nav-links');

  if (navToggle && navLinks) {
    navToggle.addEventListener('click', function () {
      var isOpen = navLinks.classList.toggle('open');
      navToggle.setAttribute('aria-expanded', isOpen);
    });

    // Close menu when a link is clicked
    navLinks.querySelectorAll('a').forEach(function (link) {
      link.addEventListener('click', function () {
        navLinks.classList.remove('open');
        navToggle.setAttribute('aria-expanded', false);
      });
    });
  }

  /* ── ACTIVE NAV LINK (scroll spy) ─────────────── */

  var sections = document.querySelectorAll('section[id], div[id]');
  var navAnchors = document.querySelectorAll('.nav-links a[href^="#"]');

  function setActiveLink() {
    var scrollPos = window.scrollY + 100;
    sections.forEach(function (section) {
      if (
        section.offsetTop <= scrollPos &&
        section.offsetTop + section.offsetHeight > scrollPos
      ) {
        navAnchors.forEach(function (a) {
          a.classList.remove('active');
          if (a.getAttribute('href') === '#' + section.id) {
            a.classList.add('active');
          }
        });
      }
    });
  }

  window.addEventListener('scroll', setActiveLink, { passive: true });

  /* ── CONTACT FORM ─────────────────────────────── */

  var form = document.getElementById('contactForm');
  if (form) {
    form.addEventListener('submit', function (e) {
      e.preventDefault();

      var name    = document.getElementById('name').value.trim();
      var email   = document.getElementById('email').value.trim();
      var message = document.getElementById('message').value.trim();

      // Simple validation
      if (!name || !email || !message) {
        alert('Please fill in your name, email, and message.');
        return;
      }

      var emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
      if (!emailRegex.test(email)) {
        alert('Please enter a valid email address.');
        return;
      }

      // ── TO ACTIVATE FORM SUBMISSION ─────────────────
      // Option 1: Formspree — replace YOUR_FORM_ID below
      //   fetch('https://formspree.io/f/YOUR_FORM_ID', {
      //     method: 'POST',
      //     headers: { 'Content-Type': 'application/json' },
      //     body: JSON.stringify({ name, email, message })
      //   });
      //
      // Option 2: Netlify Forms — add netlify attribute to <form>
      //   <form id="contactForm" netlify name="contact">
      // ────────────────────────────────────────────────

      // Show success message
      form.style.display = 'none';
      var success = document.createElement('div');
      success.className = 'form-success';
      success.style.display = 'block';
      success.innerHTML =
        '<p style="font-family: var(--font-heading); font-size: 22px; font-style: italic; color: var(--gold-light); margin-bottom: 12px;">Thank you, ' + name + '.</p>' +
        '<p style="font-size: 13px; color: rgba(245,240,232,0.6); letter-spacing: 0.5px;">Your message has been received. I\'ll be in touch within 24 hours.</p>';
      form.parentNode.insertBefore(success, form.nextSibling);
    });
  }

  /* ── PORTFOLIO ITEM HOVER ZOOM ────────────────── */
  // Already handled in CSS via .portfolio-item:hover img { transform: scale(1.04) }
  // Additional JS: add loading="lazy" to below-fold images dynamically
  var lazyImgs = document.querySelectorAll(
    '.portfolio-item img, .travel-visual img, .about-visual img'
  );
  lazyImgs.forEach(function (img) {
    if (!img.hasAttribute('loading')) {
      img.setAttribute('loading', 'lazy');
    }
  });

  /* ── FADE-IN ON SCROLL ────────────────────────── */

  var fadeEls = document.querySelectorAll(
    '.service-card, .portfolio-item, .process-step, .contact-card'
  );

  if ('IntersectionObserver' in window) {
    var observer = new IntersectionObserver(
      function (entries) {
        entries.forEach(function (entry) {
          if (entry.isIntersecting) {
            entry.target.style.opacity = '1';
            entry.target.style.transform = 'translateY(0)';
            observer.unobserve(entry.target);
          }
        });
      },
      { threshold: 0.1 }
    );

    fadeEls.forEach(function (el, i) {
      el.style.opacity = '0';
      el.style.transform = 'translateY(20px)';
      el.style.transition = 'opacity 0.6s ease ' + (i * 0.08) + 's, transform 0.6s ease ' + (i * 0.08) + 's';
      observer.observe(el);
    });
  }

})();
