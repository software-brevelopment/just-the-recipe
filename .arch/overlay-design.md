# Overlay Design Specification

## Overlay Architecture

### Modal Overlay Approach
The extension uses a modal overlay that appears on top of the existing recipe page, providing users with clean recipe data while preserving the original page context.

### User Interaction Flow
```
1. User visits recipe website
2. Extension detects recipe content automatically
3. Browser toolbar icon highlights (blue/active state)
4. User clicks toolbar icon or presses keyboard shortcut
5. Modal overlay appears with extracted recipe
6. User can:
   - Read clean recipe
   - Copy recipe to clipboard
   - Print recipe
   - Close overlay to return to original page
7. Recipe cached locally for instant re-opening
```

## Overlay Components

### Main Overlay Structure
```html
<div class="recipe-overlay" id="recipe-overlay">
  <div class="recipe-backdrop"></div>
  <div class="recipe-modal">
    <div class="recipe-header">
      <h2 class="recipe-title">Recipe Title</h2>
      <div class="recipe-actions">
        <button class="btn-copy" title="Copy Recipe">📋</button>
        <button class="btn-print" title="Print Recipe">🖨️</button>
        <button class="btn-close" title="Close (ESC)">✕</button>
      </div>
    </div>
    
    <div class="recipe-content">
      <div class="recipe-meta">
        <span class="recipe-time">⏱️ 30 mins</span>
        <span class="recipe-servings">🍽️ 4 servings</span>
        <span class="recipe-difficulty">📊 Easy</span>
      </div>
      
      <div class="recipe-section">
        <h3>Ingredients</h3>
        <ul class="ingredients-list">
          <li>1 cup flour</li>
          <li>2 eggs</li>
          <li>1/2 cup sugar</li>
        </ul>
      </div>
      
      <div class="recipe-section">
        <h3>Instructions</h3>
        <ol class="instructions-list">
          <li>Mix dry ingredients</li>
          <li>Add wet ingredients</li>
          <li>Bake at 350°F for 30 minutes</li>
        </ol>
      </div>
    </div>
  </div>
</div>
```

## CSS Design System

### Overlay Styles
```css
/* Main overlay container */
.recipe-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: 2147483647; /* Maximum z-index */
  display: flex;
  align-items: center;
  justify-content: center;
  opacity: 0;
  visibility: hidden;
  transition: opacity 0.3s ease, visibility 0.3s ease;
}

.recipe-overlay.active {
  opacity: 1;
  visibility: visible;
}

/* Backdrop */
.recipe-backdrop {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.7);
  backdrop-filter: blur(2px);
}

/* Modal container */
.recipe-modal {
  position: relative;
  background: white;
  max-width: 800px;
  max-height: 90vh;
  width: 90vw;
  border-radius: 12px;
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.3);
  overflow: hidden;
  transform: scale(0.9) translateY(20px);
  transition: transform 0.3s ease;
}

.recipe-overlay.active .recipe-modal {
  transform: scale(1) translateY(0);
}

/* Header */
.recipe-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 1.5rem 2rem;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
}

.recipe-title {
  margin: 0;
  font-size: 1.5rem;
  font-weight: 600;
  line-height: 1.2;
}

.recipe-actions {
  display: flex;
  gap: 0.5rem;
}

.btn-copy, .btn-print, .btn-close {
  background: rgba(255, 255, 255, 0.2);
  border: none;
  color: white;
  width: 40px;
  height: 40px;
  border-radius: 8px;
  cursor: pointer;
  font-size: 1.2rem;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: background 0.2s ease;
}

.btn-copy:hover, .btn-print:hover, .btn-close:hover {
  background: rgba(255, 255, 255, 0.3);
}

/* Content area */
.recipe-content {
  padding: 2rem;
  max-height: calc(90vh - 120px);
  overflow-y: auto;
}

.recipe-meta {
  display: flex;
  gap: 1rem;
  margin-bottom: 2rem;
  flex-wrap: wrap;
}

.recipe-meta span {
  background: #f7f7f7;
  padding: 0.5rem 1rem;
  border-radius: 20px;
  font-size: 0.9rem;
  font-weight: 500;
}

.recipe-section {
  margin-bottom: 2rem;
}

.recipe-section h3 {
  margin: 0 0 1rem 0;
  color: #333;
  font-size: 1.2rem;
  font-weight: 600;
}

.ingredients-list, .instructions-list {
  margin: 0;
  padding: 0;
  list-style: none;
}

.ingredients-list li {
  padding: 0.75rem 0;
  border-bottom: 1px solid #eee;
  position: relative;
  padding-left: 1.5rem;
}

.ingredients-list li:before {
  content: "•";
  position: absolute;
  left: 0;
  color: #667eea;
  font-weight: bold;
}

.instructions-list li {
  padding: 1rem 0;
  border-bottom: 1px solid #eee;
  position: relative;
  padding-left: 2.5rem;
  counter-increment: step-counter;
}

.instructions-list li:before {
  content: counter(step-counter);
  position: absolute;
  left: 0;
  top: 1rem;
  width: 1.5rem;
  height: 1.5rem;
  background: #667eea;
  color: white;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 0.8rem;
  font-weight: bold;
}

.instructions-list {
  counter-reset: step-counter;
}

/* Responsive design */
@media (max-width: 768px) {
  .recipe-modal {
    width: 95vw;
    max-height: 95vh;
  }
  
  .recipe-header {
    padding: 1rem 1.5rem;
    flex-direction: column;
    gap: 1rem;
    text-align: center;
  }
  
  .recipe-content {
    padding: 1.5rem;
  }
  
  .recipe-meta {
    justify-content: center;
  }
  
  .recipe-title {
    font-size: 1.2rem;
  }
}

/* Print styles */
@media print {
  .recipe-backdrop,
  .recipe-header,
  .recipe-actions {
    display: none !important;
  }
  
  .recipe-modal {
    box-shadow: none;
    max-width: 100%;
    max-height: none;
  }
  
  .recipe-content {
    max-height: none;
    overflow: visible;
  }
}
```

## JavaScript Implementation

### Overlay Controller
```javascript
class RecipeOverlay {
  constructor() {
    this.overlay = null;
    this.currentRecipe = null;
    this.isVisible = false;
    this.init();
  }

  init() {
    this.createOverlay();
    this.attachEventListeners();
  }

  createOverlay() {
    if (document.getElementById('recipe-overlay')) return;
    
    this.overlay = document.createElement('div');
    this.overlay.id = 'recipe-overlay';
    this.overlay.className = 'recipe-overlay';
    document.body.appendChild(this.overlay);
  }

  show(recipeData) {
    if (this.isVisible) return;
    
    this.currentRecipe = recipeData;
    this.renderRecipe(recipeData);
    this.overlay.classList.add('active');
    this.isVisible = true;
    
    // Prevent body scroll
    document.body.style.overflow = 'hidden';
    
    // Track usage
    this.trackUsage('overlay_open', {
      site: window.location.hostname,
      recipeTitle: recipeData.title
    });
  }

  hide() {
    if (!this.isVisible) return;
    
    this.overlay.classList.remove('active');
    this.isVisible = false;
    
    // Restore body scroll
    document.body.style.overflow = '';
    
    // Track usage
    this.trackUsage('overlay_close');
  }

  renderRecipe(recipe) {
    const modal = this.overlay.querySelector('.recipe-modal');
    
    modal.innerHTML = `
      <div class="recipe-header">
        <h2 class="recipe-title">${this.escapeHtml(recipe.title)}</h2>
        <div class="recipe-actions">
          <button class="btn-copy" title="Copy Recipe">📋</button>
          <button class="btn-print" title="Print Recipe">🖨️</button>
          <button class="btn-close" title="Close (ESC)">✕</button>
        </div>
      </div>
      
      <div class="recipe-content">
        ${this.renderMeta(recipe)}
        ${this.renderIngredients(recipe.ingredients)}
        ${this.renderInstructions(recipe.instructions)}
      </div>
    `;
  }

  renderMeta(recipe) {
    const meta = [];
    
    if (recipe.totalTime) {
      meta.push(`<span class="recipe-time">⏱️ ${recipe.totalTime}</span>`);
    }
    
    if (recipe.servings) {
      meta.push(`<span class="recipe-servings">🍽️ ${recipe.servings}</span>`);
    }
    
    if (recipe.difficulty) {
      meta.push(`<span class="recipe-difficulty">📊 ${recipe.difficulty}</span>`);
    }
    
    return meta.length > 0 
      ? `<div class="recipe-meta">${meta.join('')}</div>`
      : '';
  }

  renderIngredients(ingredients) {
    if (!ingredients || ingredients.length === 0) return '';
    
    const items = ingredients.map(ing => 
      `<li>${this.escapeHtml(ing.text || ing)}</li>`
    ).join('');
    
    return `
      <div class="recipe-section">
        <h3>Ingredients</h3>
        <ul class="ingredients-list">${items}</ul>
      </div>
    `;
  }

  renderInstructions(instructions) {
    if (!instructions || instructions.length === 0) return '';
    
    const items = instructions.map(inst => 
      `<li>${this.escapeHtml(inst.text || inst)}</li>`
    ).join('');
    
    return `
      <div class="recipe-section">
        <h3>Instructions</h3>
        <ol class="instructions-list">${items}</ol>
      </div>
    `;
  }

  copyRecipe() {
    const text = this.formatRecipeAsText();
    
    navigator.clipboard.writeText(text).then(() => {
      this.showToast('Recipe copied to clipboard!');
      this.trackUsage('recipe_copy');
    }).catch(() => {
      // Fallback for older browsers
      this.fallbackCopy(text);
    });
  }

  formatRecipeAsText() {
    const recipe = this.currentRecipe;
    let text = `${recipe.title}\n\n`;
    
    if (recipe.totalTime || recipe.servings) {
      text += 'Details:\n';
      if (recipe.totalTime) text += `- Time: ${recipe.totalTime}\n`;
      if (recipe.servings) text += `- Servings: ${recipe.servings}\n`;
      text += '\n';
    }
    
    text += 'Ingredients:\n';
    recipe.ingredients.forEach(ing => {
      text += `- ${ing.text || ing}\n`;
    });
    
    text += '\nInstructions:\n';
    recipe.instructions.forEach((inst, i) => {
      text += `${i + 1}. ${inst.text || inst}\n`;
    });
    
    return text;
  }

  printRecipe() {
    window.print();
    this.trackUsage('recipe_print');
  }

  attachEventListeners() {
    // Close button
    this.overlay.addEventListener('click', (e) => {
      if (e.target.classList.contains('btn-close')) {
        this.hide();
      }
      if (e.target.classList.contains('btn-copy')) {
        this.copyRecipe();
      }
      if (e.target.classList.contains('btn-print')) {
        this.printRecipe();
      }
    });

    // Close on backdrop click
    this.overlay.addEventListener('click', (e) => {
      if (e.target.classList.contains('recipe-backdrop')) {
        this.hide();
      }
    });

    // Keyboard shortcuts
    document.addEventListener('keydown', (e) => {
      if (!this.isVisible) return;
      
      if (e.key === 'Escape') {
        this.hide();
      } else if (e.ctrlKey && e.key === 'c') {
        e.preventDefault();
        this.copyRecipe();
      } else if (e.ctrlKey && e.key === 'p') {
        e.preventDefault();
        this.printRecipe();
      }
    });
  }

  escapeHtml(text) {
    const div = document.createElement('div');
    div.textContent = text;
    return div.innerHTML;
  }

  showToast(message) {
    // Simple toast notification
    const toast = document.createElement('div');
    toast.className = 'recipe-toast';
    toast.textContent = message;
    toast.style.cssText = `
      position: fixed;
      bottom: 20px;
      left: 50%;
      transform: translateX(-50%);
      background: #333;
      color: white;
      padding: 12px 24px;
      border-radius: 8px;
      z-index: 2147483648;
      opacity: 0;
      transition: opacity 0.3s ease;
    `;
    
    document.body.appendChild(toast);
    
    setTimeout(() => toast.style.opacity = '1', 10);
    setTimeout(() => {
      toast.style.opacity = '0';
      setTimeout(() => toast.remove(), 300);
    }, 2000);
  }

  trackUsage(action, data = {}) {
    // Send to Google Analytics or custom analytics
    if (typeof gtag !== 'undefined') {
      gtag('event', action, {
        event_category: 'recipe_overlay',
        ...data
      });
    }
  }
}
```

## User Experience Features

### Keyboard Shortcuts
- **ESC**: Close overlay
- **Ctrl+C**: Copy recipe to clipboard
- **Ctrl+P**: Print recipe
- **Ctrl+O**: Open overlay (when recipe detected)

### Accessibility
- **ARIA labels** for screen readers
- **Keyboard navigation** support
- **High contrast** mode compatibility
- **Focus management** within overlay

### Performance Optimizations
- **Lazy rendering** of recipe content
- **Local caching** for instant re-opening
- **Minimal DOM manipulation**
- **Efficient event delegation**

### Mobile Considerations
- **Touch-friendly** button sizes
- **Swipe gestures** for closing
- **Responsive typography**
- **Viewport height** handling

This overlay design provides a clean, modern interface that enhances the recipe viewing experience while maintaining the original page context.
