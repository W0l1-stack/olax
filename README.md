<style>
  /* Fixed container at top right */
  .language-flags-fixed {
    position: fixed;
    top: 8px; /* moved closer to top */
    right: 16px;
    z-index: 9999;
    background: #000;
    padding: 6px 10px;
    border-radius: 6px;
    display: flex;
    gap: 10px;
    align-items: center;
    box-shadow: 0 2px 8px rgba(0,0,0,0.3);
  }

  /* Each flag style */
  .language-flag {
    width: 28px;  /* smaller width */
    height: 18px; /* smaller height */
    cursor: pointer;
    border-radius: 4px;
    border: 2px solid transparent;
    transition: border-color 0.3s ease;
    transform: translateY(-3px); /* move flags up by 3px */
  }

  /* Highlight selected flag */
  .language-flag.selected {
    border-color: #fff;
  }

  /* Hover effect */
  .language-flag:hover {
    border-color: #fff;
  }
</style>

<div class="language-flags-fixed" id="languageFlags">
  <img src="https://flagcdn.com/w40/gb.png" alt="English" class="language-flag" data-lang="en" title="English">
  <img src="https://flagcdn.com/w40/es.png" alt="Spanish" class="language-flag" data-lang="es" title="Spanish">
  <img src="https://flagcdn.com/w40/it.png" alt="Italian" class="language-flag" data-lang="it" title="Italian">
</div>

<!-- Google Translate widget (hidden) -->
<div id="google_translate_element" style="display:none"></div>

<script>
  // Initialize Google Translate widget
  function googleTranslateElementInit() {
    new google.translate.TranslateElement({
      pageLanguage: 'en',
      includedLanguages: 'en,es,it',
      layout: google.translate.TranslateElement.InlineLayout.SIMPLE,
      autoDisplay: false
    }, 'google_translate_element');
  }

  // Load Google Translate script asynchronously
  (function() {
    var gtScript = document.createElement('script');
    gtScript.src = '//translate.google.com/translate_a/element.js?cb=googleTranslateElementInit';
    document.head.appendChild(gtScript);
  })();

  document.addEventListener('DOMContentLoaded', function() {
    const flagsContainer = document.getElementById('languageFlags');
    const flags = flagsContainer.querySelectorAll('.language-flag');

    const langNameMap = {
      en: 'English',
      es: 'Spanish',
      it: 'Italian'
    };

    // Highlight the selected flag
    function highlightSelected(lang) {
      flags.forEach(flag => {
        flag.classList.toggle('selected', flag.getAttribute('data-lang') === lang);
      });
    }

    // Function to click Google Translate menu item
    function clickGoogleTranslateMenu(lang) {
      const iframe = document.querySelector('iframe.goog-te-menu-frame');
      if (!iframe) {
        return false;
      }
      const innerDoc = iframe.contentDocument || iframe.contentWindow.document;
      if (!innerDoc) {
        return false;
      }
      const langItems = innerDoc.querySelectorAll('.goog-te-menu2-item span.text');
      for (let item of langItems) {
        if (item.innerText.trim() === langNameMap[lang]) {
          item.click();
          return true;
        }
      }
      return false;
    }

    // Retry clicking Google Translate menu until success or max tries
    function changeLanguage(lang, tries = 0) {
      if (tries > 10) {
        console.warn('Google Translate iframe menu not ready.');
        return;
      }
      if (!clickGoogleTranslateMenu(lang)) {
        setTimeout(() => changeLanguage(lang, tries + 1), 500);
      } else {
        localStorage.setItem('selectedLang', lang);
        highlightSelected(lang);
      }
    }

    // Attach click event to flags
    flags.forEach(flag => {
      flag.addEventListener('click', () => {
        const lang = flag.getAttribute('data-lang');
        changeLanguage(lang);
      });
    });

    // On page load, apply saved language or default to English
    const savedLang = localStorage.getItem('selectedLang') || 'en';
    highlightSelected(savedLang);
    if (savedLang !== 'en') {
      changeLanguage(savedLang);
    }
  });
</script>
