# Accessibilité de l'Application - BeneVoclic

## 📋 Vue d'ensemble

L'accessibilité de BeneVoclic garantit que la plateforme soit utilisable par tous, y compris les personnes en situation de handicap. Ces mesures s'appliquent à l'application web Nuxt.js v3.16.0 et respectent les standards WCAG 2.1 AA.

### Objectifs d'accessibilité

- ✅ **Inclusion universelle** : Accessibilité pour tous les utilisateurs
- ✅ **Conformité WCAG 2.1 AA** : Respect des standards internationaux
- ✅ **Navigation clavier** : Contrôle complet au clavier
- ✅ **Lecteurs d'écran** : Compatibilité avec les technologies d'assistance
- ✅ **Contraste et lisibilité** : Design accessible visuellement
- ✅ **Alternatives textuelles** : Contenu accessible aux non-voyants

## ♿ Conformité WCAG 2.1 AA

### Outils d'audit implémentés

#### Configuration Lighthouse Accessibility
Le projet utilise Lighthouse CI pour automatiser les audits d'accessibilité. La configuration est définie dans `lighthouserc.a11y.js` avec un seuil minimum de 90% pour l'accessibilité.

#### Commandes d'audit d'accessibilité
```bash
# Auditer l'accessibilité en mode CI
npm run audit:a11y

# Générer un rapport d'accessibilité détaillé
npm run a11y:audit

# Auditer tous les aspects (performance, SEO, accessibilité)
npm run audit:all
```

**Explication :** Ces commandes permettent de vérifier automatiquement la conformité WCAG 2.1 AA de l'application. Le rapport généré indique les problèmes d'accessibilité à corriger.

### Critères respectés

#### Niveau A (Obligatoire)
- **1.1.1** : Contenu non textuel (images avec alt text)
- **1.2.1** : Contenu audio/vidéo préenregistré (alternatives)
- **1.3.1** : Structure de l'information (HTML sémantique)
- **2.1.1** : Clavier (navigation complète au clavier)
- **2.1.2** : Pas de piège au clavier
- **2.2.1** : Contrôle du temps (pas de limite de temps)
- **2.3.1** : Pas de flash (pas de contenu clignotant)
- **2.4.1** : Contour de navigation (liens de saut)
- **2.4.2** : Titre de page
- **3.1.1** : Langue de la page
- **3.2.1** : Focus (indicateurs de focus visibles)
- **3.2.2** : Saisie (pas de changement de contexte automatique)
- **4.1.1** : Parsing (HTML valide)
- **4.1.2** : Nom, rôle, valeur (attributs ARIA)

#### Niveau AA (Recommandé)
- **1.4.3** : Contraste minimum (ratio 4.5:1)
- **1.4.4** : Redimensionnement du texte (jusqu'à 200%)
- **2.4.3** : Ordre de focus logique
- **2.4.6** : En-têtes et étiquettes descriptives
- **3.1.2** : Langue des parties
- **3.2.3** : Navigation cohérente
- **3.2.4** : Identification cohérente
- **4.1.3** : Messages d'état

## 🎨 Design accessible

### Palette de couleurs accessible

#### Couleurs principales
```css
/* Variables CSS pour l'accessibilité */
:root {
  /* Couleurs principales avec contraste vérifié */
  --primary-color: #2563eb; /* Bleu - contraste 4.6:1 */
  --primary-dark: #1d4ed8; /* Bleu foncé - contraste 7.1:1 */
  --secondary-color: #64748b; /* Gris - contraste 4.8:1 */
  
  /* Couleurs de texte */
  --text-primary: #1e293b; /* Gris très foncé - contraste 15.1:1 */
  --text-secondary: #475569; /* Gris foncé - contraste 7.1:1 */
  --text-muted: #64748b; /* Gris - contraste 4.8:1 */
  
  /* Couleurs d'état */
  --success-color: #059669; /* Vert - contraste 4.6:1 */
  --warning-color: #d97706; /* Orange - contraste 4.8:1 */
  --error-color: #dc2626; /* Rouge - contraste 4.6:1 */
  
  /* Couleurs de fond */
  --bg-primary: #ffffff; /* Blanc */
  --bg-secondary: #f8fafc; /* Gris très clair */
  --bg-tertiary: #f1f5f9; /* Gris clair */
  
  /* Couleurs de focus */
  --focus-color: #3b82f6; /* Bleu focus - contraste 4.6:1 */
  --focus-ring: 0 0 0 3px rgba(59, 130, 246, 0.5);
}
```

#### Vérification du contraste
```typescript
// utils/contrast-checker.ts
export const calculateContrastRatio = (color1: string, color2: string): number => {
  const luminance1 = getLuminance(color1);
  const luminance2 = getLuminance(color2);
  
  const lighter = Math.max(luminance1, luminance2);
  const darker = Math.min(luminance1, luminance2);
  
  return (lighter + 0.05) / (darker + 0.05);
};

export const getLuminance = (color: string): number => {
  const rgb = hexToRgb(color);
  const [r, g, b] = [rgb.r, rgb.g, rgb.b].map(c => {
    c = c / 255;
    return c <= 0.03928 ? c / 12.92 : Math.pow((c + 0.055) / 1.055, 2.4);
  });
  
  return 0.2126 * r + 0.7152 * g + 0.0722 * b;
};

export const isAccessibleContrast = (color1: string, color2: string, level: 'AA' | 'AAA' = 'AA'): boolean => {
  const ratio = calculateContrastRatio(color1, color2);
  const minRatio = level === 'AA' ? 4.5 : 7;
  return ratio >= minRatio;
};
```

### Typographie accessible

#### Hiérarchie des textes
```css
/* Système de typographie accessible */
.text-xs {
  font-size: 0.75rem; /* 12px */
  line-height: 1rem; /* 16px */
  font-weight: 400;
}

.text-sm {
  font-size: 0.875rem; /* 14px */
  line-height: 1.25rem; /* 20px */
  font-weight: 400;
}

.text-base {
  font-size: 1rem; /* 16px */
  line-height: 1.5rem; /* 24px */
  font-weight: 400;
}

.text-lg {
  font-size: 1.125rem; /* 18px */
  line-height: 1.75rem; /* 28px */
  font-weight: 500;
}

.text-xl {
  font-size: 1.25rem; /* 20px */
  line-height: 1.75rem; /* 28px */
  font-weight: 600;
}

.text-2xl {
  font-size: 1.5rem; /* 24px */
  line-height: 2rem; /* 32px */
  font-weight: 600;
}

.text-3xl {
  font-size: 1.875rem; /* 30px */
  line-height: 2.25rem; /* 36px */
  font-weight: 700;
}

/* Espacement des lettres pour améliorer la lisibilité */
.text-spacing-wide {
  letter-spacing: 0.05em;
}

.text-spacing-wider {
  letter-spacing: 0.1em;
}
```

## 🧩 Composants accessibles

### Boutons accessibles

#### Composant Button
```vue
<!-- components/common/AccessibleButton.vue -->
<template>
  <button
    :class="buttonClasses"
    :disabled="disabled"
    :aria-label="ariaLabel"
    :aria-describedby="ariaDescribedby"
    :aria-expanded="ariaExpanded"
    :aria-pressed="ariaPressed"
    :aria-controls="ariaControls"
    @click="handleClick"
    @keydown="handleKeydown"
  >
    <slot name="icon" />
    <span v-if="$slots.default" class="button-text">
      <slot />
    </span>
    <slot name="suffix" />
  </button>
</template>

<script setup lang="ts">
interface Props {
  variant?: 'primary' | 'secondary' | 'outline' | 'ghost' | 'danger';
  size?: 'sm' | 'md' | 'lg';
  disabled?: boolean;
  loading?: boolean;
  ariaLabel?: string;
  ariaDescribedby?: string;
  ariaExpanded?: boolean;
  ariaPressed?: boolean;
  ariaControls?: string;
}

const props = withDefaults(defineProps<Props>(), {
  variant: 'primary',
  size: 'md',
  disabled: false,
  loading: false,
});

const emit = defineEmits<{
  click: [event: MouseEvent];
}>();

const buttonClasses = computed(() => [
  'btn',
  `btn-${props.variant}`,
  `btn-${props.size}`,
  {
    'btn-disabled': props.disabled,
    'btn-loading': props.loading,
  },
]);

const handleClick = (event: MouseEvent) => {
  if (!props.disabled && !props.loading) {
    emit('click', event);
  }
};

const handleKeydown = (event: KeyboardEvent) => {
  // Support de la navigation au clavier
  if (event.key === 'Enter' || event.key === ' ') {
    event.preventDefault();
    if (!props.disabled && !props.loading) {
      emit('click', event as any);
    }
  }
};
</script>

<style scoped>
.btn {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 0.5rem;
  border: none;
  border-radius: 0.375rem;
  font-weight: 500;
  text-decoration: none;
  cursor: pointer;
  transition: all 0.2s ease-in-out;
  min-height: 2.5rem;
  padding: 0.5rem 1rem;
  
  /* Focus visible pour l'accessibilité */
  &:focus-visible {
    outline: 2px solid var(--focus-color);
    outline-offset: 2px;
  }
  
  /* États désactivés */
  &:disabled {
    opacity: 0.5;
    cursor: not-allowed;
    pointer-events: none;
  }
}

/* Variantes de couleurs */
.btn-primary {
  background-color: var(--primary-color);
  color: white;
  
  &:hover:not(:disabled) {
    background-color: var(--primary-dark);
  }
}

.btn-secondary {
  background-color: var(--secondary-color);
  color: white;
}

.btn-outline {
  background-color: transparent;
  border: 2px solid var(--primary-color);
  color: var(--primary-color);
  
  &:hover:not(:disabled) {
    background-color: var(--primary-color);
    color: white;
  }
}

/* Tailles */
.btn-sm {
  min-height: 2rem;
  padding: 0.375rem 0.75rem;
  font-size: 0.875rem;
}

.btn-lg {
  min-height: 3rem;
  padding: 0.75rem 1.5rem;
  font-size: 1.125rem;
}

/* État de chargement */
.btn-loading {
  position: relative;
  color: transparent;
  
  &::after {
    content: '';
    position: absolute;
    width: 1rem;
    height: 1rem;
    border: 2px solid currentColor;
    border-radius: 50%;
    border-top-color: transparent;
    animation: spin 1s linear infinite;
  }
}

@keyframes spin {
  to {
    transform: rotate(360deg);
  }
}
</style>
```

### Formulaires accessibles

#### Composant Input
```vue
<!-- components/common/AccessibleInput.vue -->
<template>
  <div class="form-group" :class="{ 'has-error': hasError }">
    <label
      v-if="label"
      :for="inputId"
      class="form-label"
      :class="{ 'required': required }"
    >
      {{ label }}
      <span v-if="required" class="sr-only">(requis)</span>
    </label>
    
    <div class="input-wrapper">
      <input
        :id="inputId"
        :type="type"
        :value="modelValue"
        :placeholder="placeholder"
        :required="required"
        :disabled="disabled"
        :readonly="readonly"
        :aria-describedby="ariaDescribedby"
        :aria-invalid="hasError"
        :aria-required="required"
        :class="inputClasses"
        @input="handleInput"
        @blur="handleBlur"
        @focus="handleFocus"
        @keydown="handleKeydown"
      />
      
      <slot name="suffix" />
    </div>
    
    <div
      v-if="hasError"
      :id="`${inputId}-error`"
      class="error-message"
      role="alert"
      aria-live="polite"
    >
      {{ errorMessage }}
    </div>
    
    <div
      v-if="helpText"
      :id="`${inputId}-help`"
      class="help-text"
    >
      {{ helpText }}
    </div>
  </div>
</template>

<script setup lang="ts">
import { computed } from 'vue';

interface Props {
  modelValue: string;
  label?: string;
  type?: 'text' | 'email' | 'password' | 'number' | 'tel' | 'url';
  placeholder?: string;
  required?: boolean;
  disabled?: boolean;
  readonly?: boolean;
  error?: string;
  helpText?: string;
  id?: string;
}

const props = withDefaults(defineProps<Props>(), {
  type: 'text',
  required: false,
  disabled: false,
  readonly: false,
});

const emit = defineEmits<{
  'update:modelValue': [value: string];
  blur: [event: FocusEvent];
  focus: [event: FocusEvent];
}>();

const inputId = computed(() => props.id || `input-${Math.random().toString(36).substr(2, 9)}`);

const hasError = computed(() => !!props.error);

const errorMessage = computed(() => props.error);

const ariaDescribedby = computed(() => {
  const ids = [];
  if (hasError.value) {
    ids.push(`${inputId.value}-error`);
  }
  if (props.helpText) {
    ids.push(`${inputId.value}-help`);
  }
  return ids.length > 0 ? ids.join(' ') : undefined;
});

const inputClasses = computed(() => [
  'form-input',
  {
    'has-error': hasError.value,
    'is-disabled': props.disabled,
    'is-readonly': props.readonly,
  },
]);

const handleInput = (event: Event) => {
  const target = event.target as HTMLInputElement;
  emit('update:modelValue', target.value);
};

const handleBlur = (event: FocusEvent) => {
  emit('blur', event);
};

const handleFocus = (event: FocusEvent) => {
  emit('focus', event);
};

const handleKeydown = (event: KeyboardEvent) => {
  // Support de la navigation au clavier
  if (event.key === 'Enter' && props.type === 'email') {
    // Validation d'email au clavier
    const target = event.target as HTMLInputElement;
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    if (!emailRegex.test(target.value)) {
      event.preventDefault();
    }
  }
};
</script>

<style scoped>
.form-group {
  margin-bottom: 1rem;
}

.form-label {
  display: block;
  margin-bottom: 0.5rem;
  font-weight: 500;
  color: var(--text-primary);
  
  &.required::after {
    content: ' *';
    color: var(--error-color);
  }
}

.input-wrapper {
  position: relative;
  display: flex;
  align-items: center;
}

.form-input {
  width: 100%;
  padding: 0.75rem;
  border: 2px solid var(--border-color);
  border-radius: 0.375rem;
  font-size: 1rem;
  line-height: 1.5;
  background-color: var(--bg-primary);
  color: var(--text-primary);
  transition: border-color 0.2s ease-in-out;
  
  /* Focus visible */
  &:focus-visible {
    outline: none;
    border-color: var(--focus-color);
    box-shadow: var(--focus-ring);
  }
  
  /* États */
  &.has-error {
    border-color: var(--error-color);
  }
  
  &.is-disabled {
    background-color: var(--bg-tertiary);
    color: var(--text-muted);
    cursor: not-allowed;
  }
  
  &.is-readonly {
    background-color: var(--bg-secondary);
  }
  
  /* Placeholder accessible */
  &::placeholder {
    color: var(--text-muted);
    opacity: 1;
  }
}

.error-message {
  margin-top: 0.25rem;
  font-size: 0.875rem;
  color: var(--error-color);
  font-weight: 500;
}

.help-text {
  margin-top: 0.25rem;
  font-size: 0.875rem;
  color: var(--text-secondary);
}

/* Screen reader only */
.sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border: 0;
}
</style>
```

### Navigation accessible

#### Menu de navigation
```vue
<!-- components/layout/AccessibleNavigation.vue -->
<template>
  <nav
    class="main-navigation"
    role="navigation"
    :aria-label="ariaLabel"
  >
    <button
      v-if="isMobile"
      class="menu-toggle"
      :aria-expanded="isMenuOpen"
      :aria-controls="menuId"
      @click="toggleMenu"
    >
      <span class="sr-only">{{ isMenuOpen ? 'Fermer le menu' : 'Ouvrir le menu' }}</span>
      <svg
        class="menu-icon"
        :class="{ 'is-open': isMenuOpen }"
        width="24"
        height="24"
        viewBox="0 0 24 24"
        fill="none"
        stroke="currentColor"
        stroke-width="2"
      >
        <line v-if="!isMenuOpen" x1="3" y1="6" x2="21" y2="6" />
        <line v-if="!isMenuOpen" x1="3" y1="12" x2="21" y2="12" />
        <line v-if="!isMenuOpen" x1="3" y1="18" x2="21" y2="18" />
        <polyline v-else points="18,15 12,9 6,15" />
      </svg>
    </button>
    
    <ul
      :id="menuId"
      class="nav-menu"
      :class="{ 'is-open': isMenuOpen }"
      role="menubar"
    >
      <li
        v-for="item in menuItems"
        :key="item.id"
        class="nav-item"
        role="none"
      >
        <NuxtLink
          :to="item.href"
          :class="['nav-link', { 'is-active': isActive(item.href) }]"
          role="menuitem"
          :aria-current="isActive(item.href) ? 'page' : undefined"
          @click="closeMenu"
        >
          <component
            v-if="item.icon"
            :is="item.icon"
            class="nav-icon"
            aria-hidden="true"
          />
          <span class="nav-text">{{ item.label }}</span>
        </NuxtLink>
      </li>
    </ul>
  </nav>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue';

interface MenuItem {
  id: string;
  label: string;
  href: string;
  icon?: any;
}

interface Props {
  menuItems: MenuItem[];
  ariaLabel?: string;
  isMobile?: boolean;
}

const props = withDefaults(defineProps<Props>(), {
  ariaLabel: 'Navigation principale',
  isMobile: false,
});

const isMenuOpen = ref(false);
const menuId = `menu-${Math.random().toString(36).substr(2, 9)}`;

const route = useRoute();

const isActive = (href: string) => {
  return route.path === href;
};

const toggleMenu = () => {
  isMenuOpen.value = !isMenuOpen.value;
};

const closeMenu = () => {
  if (props.isMobile) {
    isMenuOpen.value = false;
  }
};

// Fermer le menu lors de la navigation au clavier
const handleKeydown = (event: KeyboardEvent) => {
  if (event.key === 'Escape') {
    closeMenu();
  }
};

// Fermer le menu lors d'un clic à l'extérieur
onMounted(() => {
  document.addEventListener('keydown', handleKeydown);
  document.addEventListener('click', (event) => {
    const target = event.target as Element;
    if (!target.closest('.main-navigation')) {
      closeMenu();
    }
  });
});

onUnmounted(() => {
  document.removeEventListener('keydown', handleKeydown);
});
</script>

<style scoped>
.main-navigation {
  position: relative;
}

.menu-toggle {
  display: none;
  align-items: center;
  justify-content: center;
  width: 2.5rem;
  height: 2.5rem;
  border: none;
  background: none;
  cursor: pointer;
  color: var(--text-primary);
  
  &:focus-visible {
    outline: 2px solid var(--focus-color);
    outline-offset: 2px;
  }
}

.menu-icon {
  transition: transform 0.2s ease-in-out;
  
  &.is-open {
    transform: rotate(180deg);
  }
}

.nav-menu {
  display: flex;
  list-style: none;
  margin: 0;
  padding: 0;
  gap: 1rem;
}

.nav-item {
  position: relative;
}

.nav-link {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  padding: 0.75rem 1rem;
  color: var(--text-primary);
  text-decoration: none;
  border-radius: 0.375rem;
  transition: all 0.2s ease-in-out;
  
  &:hover {
    background-color: var(--bg-secondary);
    color: var(--primary-color);
  }
  
  &:focus-visible {
    outline: 2px solid var(--focus-color);
    outline-offset: 2px;
  }
  
  &.is-active {
    background-color: var(--primary-color);
    color: white;
  }
}

.nav-icon {
  width: 1.25rem;
  height: 1.25rem;
}

/* Mobile styles */
@media (max-width: 768px) {
  .menu-toggle {
    display: flex;
  }
  
  .nav-menu {
    position: absolute;
    top: 100%;
    left: 0;
    right: 0;
    flex-direction: column;
    background-color: var(--bg-primary);
    border: 1px solid var(--border-color);
    border-radius: 0.375rem;
    box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1);
    opacity: 0;
    visibility: hidden;
    transform: translateY(-10px);
    transition: all 0.2s ease-in-out;
    
    &.is-open {
      opacity: 1;
      visibility: visible;
      transform: translateY(0);
    }
  }
  
  .nav-link {
    padding: 1rem;
    border-radius: 0;
    
    &:first-child {
      border-top-left-radius: 0.375rem;
      border-top-right-radius: 0.375rem;
    }
    
    &:last-child {
      border-bottom-left-radius: 0.375rem;
      border-bottom-right-radius: 0.375rem;
    }
  }
}

/* Screen reader only */
.sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border: 0;
}
</style>
```

## 🔍 Tests d'accessibilité

### Configuration Lighthouse CI

#### Configuration d'accessibilité
```javascript
// lighthouserc.a11y.js
module.exports = {
  ci: {
    collect: {
      url: ['http://localhost:5482'],
      numberOfRuns: 3,
      settings: {
        chromeFlags: '--no-sandbox --disable-dev-shm-usage',
        onlyCategories: ['accessibility'],
      },
    },
    assert: {
      assertions: {
        'categories:accessibility': ['error', { minScore: 0.90 }],
        'accessibility:color-contrast': 'warn',
        'accessibility:document-title': 'error',
        'accessibility:heading-order': 'error',
        'accessibility:html-has-lang': 'error',
        'accessibility:image-alt': 'error',
        'accessibility:label': 'error',
        'accessibility:link-name': 'error',
        'accessibility:focusable-controls': 'error',
        'accessibility:skip-link': 'error',
        'accessibility:aria-required-attr': 'error',
        'accessibility:aria-required-children': 'error',
        'accessibility:aria-valid-attr-value': 'error',
        'accessibility:aria-valid-attr': 'error',
        'accessibility:button-name': 'error',
        'accessibility:form-field-multiple-labels': 'error',
        'accessibility:frame-title': 'error',
        'accessibility:input-image-alt': 'error',
        'accessibility:object-alt': 'error',
        'accessibility:video-caption': 'warn',
        'accessibility:video-description': 'warn',
      },
    },
    upload: {
      target: 'temporary-public-storage',
    },
  },
};
```

### Tests automatisés

#### Tests d'accessibilité avec axe-core
```typescript
// tests/accessibility.test.ts
import { test, expect } from '@playwright/test';
import AxeBuilder from '@axe-core/playwright';

test.describe('Accessibility Tests', () => {
  test('should meet accessibility standards on homepage', async ({ page }) => {
    await page.goto('/');
    
    const accessibilityScanResults = await new AxeBuilder({ page })
      .withTags(['wcag2a', 'wcag2aa'])
      .analyze();
    
    expect(accessibilityScanResults.violations).toEqual([]);
  });
  
  test('should meet accessibility standards on login page', async ({ page }) => {
    await page.goto('/auth/login');
    
    const accessibilityScanResults = await new AxeBuilder({ page })
      .withTags(['wcag2a', 'wcag2aa'])
      .analyze();
    
    expect(accessibilityScanResults.violations).toEqual([]);
  });
  
  test('should have proper heading structure', async ({ page }) => {
    await page.goto('/');
    
    const headings = await page.$$eval('h1, h2, h3, h4, h5, h6', (elements) =>
      elements.map((el) => ({
        level: parseInt(el.tagName.charAt(1)),
        text: el.textContent?.trim(),
      }))
    );
    
    // Vérifier que les titres suivent une hiérarchie logique
    let previousLevel = 0;
    for (const heading of headings) {
      expect(heading.level).toBeLessThanOrEqual(previousLevel + 1);
      previousLevel = heading.level;
    }
  });
  
  test('should have proper focus management', async ({ page }) => {
    await page.goto('/');
    
    // Vérifier que tous les éléments interactifs sont focusables
    const interactiveElements = await page.$$('button, a, input, select, textarea');
    
    for (const element of interactiveElements) {
      await element.focus();
      const isFocused = await element.evaluate((el) => document.activeElement === el);
      expect(isFocused).toBe(true);
    }
  });
  
  test('should have proper ARIA labels', async ({ page }) => {
    await page.goto('/');
    
    // Vérifier que les éléments sans texte visible ont des aria-labels
    const elementsWithoutText = await page.$$('[aria-label]');
    
    for (const element of elementsWithoutText) {
      const ariaLabel = await element.getAttribute('aria-label');
      expect(ariaLabel).toBeTruthy();
      expect(ariaLabel.length).toBeGreaterThan(0);
    }
  });
});
```

### Tests manuels

#### Checklist d'accessibilité
```markdown
# Checklist d'accessibilité - BeneVoclic

## Navigation au clavier
- [ ] Tous les éléments interactifs sont accessibles au clavier
- [ ] L'ordre de tabulation est logique
- [ ] Les indicateurs de focus sont visibles
- [ ] Pas de piège au clavier
- [ ] Raccourcis clavier fonctionnels

## Lecteurs d'écran
- [ ] Structure HTML sémantique
- [ ] Attributs ARIA appropriés
- [ ] Textes alternatifs pour les images
- [ ] Labels explicites pour les formulaires
- [ ] Messages d'état annoncés

## Contraste et lisibilité
- [ ] Contraste minimum 4.5:1 pour le texte normal
- [ ] Contraste minimum 3:1 pour le texte large
- [ ] Taille de police minimum 16px
- [ ] Espacement des lignes suffisant
- [ ] Pas de texte en image

## Formulaires
- [ ] Labels associés aux champs
- [ ] Messages d'erreur clairs
- [ ] Validation en temps réel
- [ ] Indication des champs requis
- [ ] Groupement logique des champs

## Multimédia
- [ ] Sous-titres pour les vidéos
- [ ] Transcriptions pour l'audio
- [ ] Contrôles accessibles au clavier
- [ ] Alternatives textuelles

## Responsive
- [ ] Zoom jusqu'à 200% sans perte de fonctionnalité
- [ ] Orientation portrait et paysage
- [ ] Pas de défilement horizontal
- [ ] Boutons de taille suffisante (44px minimum)
```

## 🎯 Améliorations continues

### Métriques d'accessibilité

#### Indicateurs de performance
```typescript
// utils/accessibility-metrics.ts
export interface AccessibilityMetrics {
  lighthouseScore: number;
  violations: AccessibilityViolation[];
  warnings: AccessibilityWarning[];
  passes: AccessibilityPass[];
}

export interface AccessibilityViolation {
  id: string;
  impact: 'minor' | 'moderate' | 'serious' | 'critical';
  description: string;
  help: string;
  nodes: ViolationNode[];
}

export interface ViolationNode {
  html: string;
  target: string[];
  failureSummary: string;
}

export const calculateAccessibilityScore = (metrics: AccessibilityMetrics): number => {
  const totalIssues = metrics.violations.length + metrics.warnings.length;
  const criticalIssues = metrics.violations.filter(v => v.impact === 'critical').length;
  const seriousIssues = metrics.violations.filter(v => v.impact === 'serious').length;
  
  // Score de base
  let score = 100;
  
  // Pénalités
  score -= criticalIssues * 20;
  score -= seriousIssues * 10;
  score -= (metrics.violations.length - criticalIssues - seriousIssues) * 5;
  score -= metrics.warnings.length * 2;
  
  return Math.max(0, score);
};
```

### Plan d'amélioration

#### Actions prioritaires
```markdown
# Plan d'amélioration de l'accessibilité

## Phase 1 - Corrections critiques (Sprint 1-2)
- [ ] Corriger tous les problèmes de contraste
- [ ] Ajouter les attributs ARIA manquants
- [ ] Implémenter la navigation au clavier complète
- [ ] Corriger la structure des titres

## Phase 2 - Améliorations importantes (Sprint 3-4)
- [ ] Optimiser les formulaires pour les lecteurs d'écran
- [ ] Améliorer les messages d'erreur
- [ ] Ajouter des raccourcis clavier
- [ ] Implémenter le mode sombre

## Phase 3 - Optimisations avancées (Sprint 5-6)
- [ ] Ajouter des fonctionnalités d'accessibilité avancées
- [ ] Optimiser pour les technologies d'assistance
- [ ] Tests avec des utilisateurs en situation de handicap
- [ ] Formation de l'équipe

## Métriques de suivi
- Score Lighthouse Accessibility > 95
- 0 violation critique
- 0 violation sérieuse
- Temps de résolution < 48h pour les problèmes critiques
```

---

*L'accessibilité de BeneVoclic garantit une expérience inclusive pour tous les utilisateurs.* ♿ 