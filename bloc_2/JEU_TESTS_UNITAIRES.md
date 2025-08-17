# Jeu de Tests Unitaires - BeneVoclic

## 📋 Vue d'ensemble

Le jeu de tests unitaires de BeneVoclic assure la qualité et la fiabilité du code en validant le comportement de chaque composant de manière isolée. Les tests couvrent l'API NestJS v11.0.7 et l'application web Nuxt.js v3.16.0.

### Objectifs des tests unitaires

- ✅ **Validation du comportement** : Vérification de la logique métier
- ✅ **Détection précoce** : Identification rapide des régressions
- ✅ **Documentation vivante** : Tests comme spécifications
- ✅ **Refactoring sécurisé** : Confiance pour les modifications
- ✅ **Couverture de code** : Validation de tous les chemins d'exécution

## 🧪 Configuration des tests

### Backend - Jest (NestJS) - Implémenté

#### Configuration Jest
Le backend utilise Jest pour les tests unitaires et d'intégration. La configuration est définie dans `jest.config.js` avec :
- Support TypeScript complet
- Couverture de code configurée
- Tests d'intégration séparés
- Mapping des alias de modules

#### Commandes de test backend
```bash
# Exécuter tous les tests
npm test

# Tests unitaires uniquement
npm run test:unit

# Tests d'intégration
npm run test:integration

# Tests en mode watch (développement)
npm run test:watch

# Générer un rapport de couverture
npm run test:cov

# Tests pour CI/CD
npm run test:ci
```

**Explication :** Ces commandes permettent d'exécuter différents types de tests selon les besoins. Les tests unitaires vérifient les fonctions individuelles, tandis que les tests d'intégration valident les interactions entre composants.

### Backend - Jest (NestJS)
```
module.exports = {
  moduleFileExtensions: ['js', 'json', 'ts'],
  rootDir: 'src',
  testRegex: '.*\\.spec\\.ts$',
  transform: {
    '^.+\\.(t|j)s$': 'ts-jest',
  },
  collectCoverageFrom: [
    '**/*.(t|j)s',
    '!**/*.spec.ts',
    '!**/*.e2e-spec.ts',
    '!**/index.ts',
    '!**/*.module.ts',
  ],
  coverageDirectory: '../coverage',
  coverageReporters: ['text', 'lcov', 'html'],
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80,
    },
  },
  testEnvironment: 'node',
  setupFilesAfterEnv: ['<rootDir>/../test/setup.ts'],
  testTimeout: 10000,
};
```

### Frontend - Vitest (Nuxt.js) - Implémenté

#### Configuration Vitest
Le frontend utilise Vitest pour les tests unitaires et de composants. La configuration est définie dans `vitest.config.ts` avec :
- Environnement DOM simulé (happy-dom)
- Couverture de code ciblée sur les composants
- Seuils de qualité configurés (70% minimum)
- Exclusion des fichiers non testables

#### Commandes de test frontend
```bash
# Exécuter tous les tests
npm test

# Tests des composants uniquement
npm run test:components

# Exécuter tous les tests avec rapport
npm run test:all

# Générer un rapport de couverture
npm run test:coverage

# Tests d'accessibilité
npm run test:a11y
```

**Explication :** Vitest est plus rapide que Jest pour les tests frontend et offre une meilleure intégration avec Vue.js. Les tests de composants vérifient le rendu et les interactions utilisateur.

### Frontend - Vitest (Nuxt.js)
```
import { defineConfig } from 'vitest/config'
import vue from '@vitejs/plugin-vue'

export default defineConfig({
  plugins: [vue()],
  test: {
    environment: 'happy-dom',
    globals: true,
    setupFiles: ['./test/setup.ts'],
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html', 'lcov'],
      exclude: [
        'node_modules/',
        'test/',
        '**/*.d.ts',
        '**/*.config.*',
        '**/coverage/**',
        '**/dist/**',
        '**/.nuxt/**',
        '**/.output/**'
      ],
      thresholds: {
        global: {
          branches: 70,
          functions: 70,
          lines: 70,
          statements: 70,
        },
      },
    },
    testTimeout: 10000,
  },
  resolve: {
    alias: {
      '@': '/src',
      '~': '/src',
    },
  },
})
```

## 🔧 Tests Backend - Services

### Test du service utilisateur

```typescript
// user.service.spec.ts
import { Test, TestingModule } from '@nestjs/testing';
import { getModelToken } from '@nestjs/mongoose';
import { UserService } from './user.service';
import { User } from './schemas/user.schema';
import { CreateUserDto } from './dto/create-user.dto';
import { UpdateUserDto } from './dto/update-user.dto';
import { ConflictException, NotFoundException } from '@nestjs/common';

describe('UserService', () => {
  let service: UserService;
  let userModel: any;

  const mockUserModel = {
    findOne: jest.fn(),
    findById: jest.fn(),
    findByIdAndUpdate: jest.fn(),
    findByIdAndDelete: jest.fn(),
    save: jest.fn(),
  };

  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      providers: [
        UserService,
        {
          provide: getModelToken(User.name),
          useValue: mockUserModel,
        },
      ],
    }).compile();

    service = module.get<UserService>(UserService);
    userModel = module.get(getModelToken(User.name));
  });

  afterEach(() => {
    jest.clearAllMocks();
  });

  describe('create', () => {
    it('should create a new user successfully', async () => {
      const createUserDto: CreateUserDto = {
        firstName: 'John',
        lastName: 'Doe',
        email: 'john@example.com',
        role: 'volunteer',
      };

      const savedUser = {
        _id: 'user123',
        ...createUserDto,
        createdAt: new Date(),
        updatedAt: new Date(),
      };

      userModel.findOne.mockResolvedValue(null);
      userModel.save.mockResolvedValue(savedUser);

      const result = await service.create(createUserDto);

      expect(userModel.findOne).toHaveBeenCalledWith({
        email: createUserDto.email,
      });
      expect(result).toEqual(savedUser);
    });

    it('should throw ConflictException if user already exists', async () => {
      const createUserDto: CreateUserDto = {
        firstName: 'John',
        lastName: 'Doe',
        email: 'existing@example.com',
        role: 'volunteer',
      };

      userModel.findOne.mockResolvedValue({ email: 'existing@example.com' });

      await expect(service.create(createUserDto)).rejects.toThrow(
        ConflictException,
      );
    });
  });

  describe('findById', () => {
    it('should return user if found', async () => {
      const userId = 'user123';
      const mockUser = {
        _id: userId,
        firstName: 'John',
        lastName: 'Doe',
        email: 'john@example.com',
        role: 'volunteer',
      };

      userModel.findById.mockReturnValue({
        exec: jest.fn().mockResolvedValue(mockUser),
      });

      const result = await service.findById(userId);

      expect(userModel.findById).toHaveBeenCalledWith(userId);
      expect(result).toEqual(mockUser);
    });

    it('should throw NotFoundException if user not found', async () => {
      const userId = 'nonexistent';

      userModel.findById.mockReturnValue({
        exec: jest.fn().mockResolvedValue(null),
      });

      await expect(service.findById(userId)).rejects.toThrow(
        NotFoundException,
      );
    });
  });

  describe('update', () => {
    it('should update user successfully', async () => {
      const userId = 'user123';
      const updateUserDto: UpdateUserDto = {
        firstName: 'Jane',
        lastName: 'Smith',
      };

      const updatedUser = {
        _id: userId,
        ...updateUserDto,
        email: 'john@example.com',
        role: 'volunteer',
      };

      userModel.findByIdAndUpdate.mockReturnValue({
        exec: jest.fn().mockResolvedValue(updatedUser),
      });

      const result = await service.update(userId, updateUserDto);

      expect(userModel.findByIdAndUpdate).toHaveBeenCalledWith(
        userId,
        updateUserDto,
        { new: true },
      );
      expect(result).toEqual(updatedUser);
    });

    it('should throw NotFoundException if user not found', async () => {
      const userId = 'nonexistent';
      const updateUserDto: UpdateUserDto = {
        firstName: 'Jane',
      };

      userModel.findByIdAndUpdate.mockReturnValue({
        exec: jest.fn().mockResolvedValue(null),
      });

      await expect(service.update(userId, updateUserDto)).rejects.toThrow(
        NotFoundException,
      );
    });
  });

  describe('remove', () => {
    it('should delete user successfully', async () => {
      const userId = 'user123';
      const deletedUser = {
        _id: userId,
        firstName: 'John',
        lastName: 'Doe',
      };

      userModel.findByIdAndDelete.mockReturnValue({
        exec: jest.fn().mockResolvedValue(deletedUser),
      });

      await service.remove(userId);

      expect(userModel.findByIdAndDelete).toHaveBeenCalledWith(userId);
    });

    it('should throw NotFoundException if user not found', async () => {
      const userId = 'nonexistent';

      userModel.findByIdAndDelete.mockReturnValue({
        exec: jest.fn().mockResolvedValue(null),
      });

      await expect(service.remove(userId)).rejects.toThrow(
        NotFoundException,
      );
    });
  });
});
```

### Test du service d'authentification

```typescript
// auth.service.spec.ts
import { Test, TestingModule } from '@nestjs/testing';
import { getModelToken } from '@nestjs/mongoose';
import { JwtService } from '@nestjs/jwt';
import { AuthService } from './auth.service';
import { User } from '../users/schemas/user.schema';
import { CreateUserDto } from '../users/dto/create-user.dto';
import { ConflictException, UnauthorizedException } from '@nestjs/common';
import * as bcrypt from 'bcrypt';

describe('AuthService', () => {
  let service: AuthService;
  let userModel: any;
  let jwtService: JwtService;

  const mockUserModel = {
    findOne: jest.fn(),
    save: jest.fn(),
  };

  const mockJwtService = {
    sign: jest.fn(),
  };

  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      providers: [
        AuthService,
        {
          provide: getModelToken(User.name),
          useValue: mockUserModel,
        },
        {
          provide: JwtService,
          useValue: mockJwtService,
        },
      ],
    }).compile();

    service = module.get<AuthService>(AuthService);
    userModel = module.get(getModelToken(User.name));
    jwtService = module.get<JwtService>(JwtService);
  });

  afterEach(() => {
    jest.clearAllMocks();
  });

  describe('validateUser', () => {
    it('should return user if credentials are valid', async () => {
      const email = 'john@example.com';
      const password = 'password123';
      const hashedPassword = await bcrypt.hash(password, 10);

      const mockUser = {
        _id: 'user123',
        email,
        password: hashedPassword,
        firstName: 'John',
        lastName: 'Doe',
        role: 'volunteer',
        toObject: jest.fn().mockReturnValue({
          _id: 'user123',
          email,
          password: hashedPassword,
          firstName: 'John',
          lastName: 'Doe',
          role: 'volunteer',
        }),
      };

      userModel.findOne.mockResolvedValue(mockUser);

      const result = await service.validateUser(email, password);

      expect(userModel.findOne).toHaveBeenCalledWith({ email });
      expect(result).toEqual({
        _id: 'user123',
        email,
        firstName: 'John',
        lastName: 'Doe',
        role: 'volunteer',
      });
    });

    it('should return null if user not found', async () => {
      const email = 'nonexistent@example.com';
      const password = 'password123';

      userModel.findOne.mockResolvedValue(null);

      const result = await service.validateUser(email, password);

      expect(result).toBeNull();
    });

    it('should return null if password is invalid', async () => {
      const email = 'john@example.com';
      const password = 'wrongpassword';
      const hashedPassword = await bcrypt.hash('correctpassword', 10);

      const mockUser = {
        _id: 'user123',
        email,
        password: hashedPassword,
        toObject: jest.fn().mockReturnValue({
          _id: 'user123',
          email,
          password: hashedPassword,
        }),
      };

      userModel.findOne.mockResolvedValue(mockUser);

      const result = await service.validateUser(email, password);

      expect(result).toBeNull();
    });
  });

  describe('login', () => {
    it('should return access token and user data', async () => {
      const user = {
        _id: 'user123',
        email: 'john@example.com',
        firstName: 'John',
        lastName: 'Doe',
        role: 'volunteer',
      };

      const token = 'jwt-token';
      mockJwtService.sign.mockReturnValue(token);

      const result = await service.login(user);

      expect(jwtService.sign).toHaveBeenCalledWith({
        email: user.email,
        sub: user._id,
        role: user.role,
      });
      expect(result).toEqual({
        access_token: token,
        user: {
          id: user._id,
          email: user.email,
          firstName: user.firstName,
          lastName: user.lastName,
          role: user.role,
        },
      });
    });
  });

  describe('register', () => {
    it('should register new user successfully', async () => {
      const createUserDto: CreateUserDto = {
        firstName: 'John',
        lastName: 'Doe',
        email: 'john@example.com',
        password: 'password123',
        role: 'volunteer',
      };

      userModel.findOne.mockResolvedValue(null);

      const savedUser = {
        _id: 'user123',
        ...createUserDto,
        password: 'hashedpassword',
        toObject: jest.fn().mockReturnValue({
          _id: 'user123',
          ...createUserDto,
          password: 'hashedpassword',
        }),
      };

      userModel.save.mockResolvedValue(savedUser);

      const result = await service.register(createUserDto);

      expect(userModel.findOne).toHaveBeenCalledWith({
        email: createUserDto.email,
      });
      expect(result).toEqual({
        _id: 'user123',
        firstName: 'John',
        lastName: 'Doe',
        email: 'john@example.com',
        role: 'volunteer',
      });
    });

    it('should throw ConflictException if user already exists', async () => {
      const createUserDto: CreateUserDto = {
        firstName: 'John',
        lastName: 'Doe',
        email: 'existing@example.com',
        password: 'password123',
        role: 'volunteer',
      };

      userModel.findOne.mockResolvedValue({ email: 'existing@example.com' });

      await expect(service.register(createUserDto)).rejects.toThrow(
        ConflictException,
      );
    });
  });
});
```

## 🧪 Tests Frontend - Composables

### Test du composable useEvents

```typescript
// useEvents.test.ts
import { describe, it, expect, vi, beforeEach } from 'vitest'
import { useEvents } from '@/composables/useEvents'

// Mock Nuxt composables
vi.mock('#app', () => ({
  useNuxtApp: () => ({
    $api: {
      get: vi.fn(),
      post: vi.fn(),
      put: vi.fn(),
      delete: vi.fn(),
    },
  }),
}))

describe('useEvents', () => {
  let mockApi: any

  beforeEach(async () => {
    const { useNuxtApp } = await import('#app')
    mockApi = useNuxtApp().$api
    vi.clearAllMocks()
  })

  describe('fetchEvents', () => {
    it('should fetch events successfully', async () => {
      const mockEvents = [
        {
          id: '1',
          title: 'Event 1',
          description: 'Description 1',
          date: '2024-01-01',
        },
        {
          id: '2',
          title: 'Event 2',
          description: 'Description 2',
          date: '2024-01-02',
        },
      ]

      const mockResponse = {
        data: {
          events: mockEvents,
          totalPages: 1,
          currentPage: 1,
          totalItems: 2,
        },
      }

      mockApi.get.mockResolvedValue(mockResponse)

      const { fetchEvents, loading, error } = useEvents()

      const result = await fetchEvents({
        page: 1,
        filters: { category: 'volunteer' },
      })

      expect(mockApi.get).toHaveBeenCalledWith('/events', {
        params: {
          page: 1,
          filters: { category: 'volunteer' },
        },
      })
      expect(result).toEqual(mockResponse.data)
      expect(loading.value).toBe(false)
      expect(error.value).toBeNull()
    })

    it('should handle fetch errors', async () => {
      const mockError = new Error('Network error')
      mockApi.get.mockRejectedValue(mockError)

      const { fetchEvents, loading, error } = useEvents()

      await expect(
        fetchEvents({ page: 1 }),
      ).rejects.toThrow('Network error')

      expect(loading.value).toBe(false)
      expect(error.value).toBe('Erreur lors du chargement des événements')
    })
  })

  describe('createEvent', () => {
    it('should create event successfully', async () => {
      const eventData = {
        title: 'New Event',
        description: 'New Description',
        date: '2024-01-01',
      }

      const mockResponse = {
        data: {
          id: '3',
          ...eventData,
        },
      }

      mockApi.post.mockResolvedValue(mockResponse)

      const { createEvent, loading, error } = useEvents()

      const result = await createEvent(eventData)

      expect(mockApi.post).toHaveBeenCalledWith('/events', eventData)
      expect(result).toEqual(mockResponse.data)
      expect(loading.value).toBe(false)
      expect(error.value).toBeNull()
    })

    it('should handle creation errors', async () => {
      const eventData = {
        title: 'New Event',
        description: 'New Description',
        date: '2024-01-01',
      }

      const mockError = new Error('Validation error')
      mockApi.post.mockRejectedValue(mockError)

      const { createEvent, loading, error } = useEvents()

      await expect(createEvent(eventData)).rejects.toThrow('Validation error')

      expect(loading.value).toBe(false)
      expect(error.value).toBe('Erreur lors de la création de l\'événement')
    })
  })

  describe('updateEvent', () => {
    it('should update event successfully', async () => {
      const eventId = '1'
      const eventData = {
        title: 'Updated Event',
        description: 'Updated Description',
      }

      const mockResponse = {
        data: {
          id: eventId,
          ...eventData,
        },
      }

      mockApi.put.mockResolvedValue(mockResponse)

      const { updateEvent, loading, error } = useEvents()

      const result = await updateEvent(eventId, eventData)

      expect(mockApi.put).toHaveBeenCalledWith(`/events/${eventId}`, eventData)
      expect(result).toEqual(mockResponse.data)
      expect(loading.value).toBe(false)
      expect(error.value).toBeNull()
    })
  })

  describe('deleteEvent', () => {
    it('should delete event successfully', async () => {
      const eventId = '1'

      mockApi.delete.mockResolvedValue({})

      const { deleteEvent, loading, error } = useEvents()

      await deleteEvent(eventId)

      expect(mockApi.delete).toHaveBeenCalledWith(`/events/${eventId}`)
      expect(loading.value).toBe(false)
      expect(error.value).toBeNull()
    })
  })
})
```

### Test du composable useAuth

```typescript
// useAuth.test.ts
import { describe, it, expect, vi, beforeEach } from 'vitest'
import { useAuth } from '@/composables/useAuth'

// Mock Pinia store
vi.mock('pinia', () => ({
  defineStore: vi.fn(),
  storeToRefs: vi.fn(() => ({
    user: { value: null },
    isAuthenticated: { value: false },
  })),
}))

// Mock Nuxt composables
vi.mock('#app', () => ({
  useNuxtApp: () => ({
    $api: {
      post: vi.fn(),
    },
  }),
}))

describe('useAuth', () => {
  let mockApi: any

  beforeEach(async () => {
    const { useNuxtApp } = await import('#app')
    mockApi = useNuxtApp().$api
    vi.clearAllMocks()
  })

  describe('login', () => {
    it('should login successfully', async () => {
      const credentials = {
        email: 'john@example.com',
        password: 'password123',
      }

      const mockResponse = {
        data: {
          access_token: 'jwt-token',
          user: {
            id: 'user123',
            email: 'john@example.com',
            firstName: 'John',
            lastName: 'Doe',
            role: 'volunteer',
          },
        },
      }

      mockApi.post.mockResolvedValue(mockResponse)

      const { login, loading, error } = useAuth()

      const result = await login(credentials)

      expect(mockApi.post).toHaveBeenCalledWith('/auth/login', credentials)
      expect(result).toEqual(mockResponse.data)
      expect(loading.value).toBe(false)
      expect(error.value).toBeNull()
    })

    it('should handle login errors', async () => {
      const credentials = {
        email: 'john@example.com',
        password: 'wrongpassword',
      }

      const mockError = new Error('Invalid credentials')
      mockApi.post.mockRejectedValue(mockError)

      const { login, loading, error } = useAuth()

      await expect(login(credentials)).rejects.toThrow('Invalid credentials')

      expect(loading.value).toBe(false)
      expect(error.value).toBe('Erreur lors de la connexion')
    })
  })

  describe('register', () => {
    it('should register successfully', async () => {
      const userData = {
        firstName: 'John',
        lastName: 'Doe',
        email: 'john@example.com',
        password: 'password123',
        role: 'volunteer',
      }

      const mockResponse = {
        data: {
          id: 'user123',
          ...userData,
        },
      }

      mockApi.post.mockResolvedValue(mockResponse)

      const { register, loading, error } = useAuth()

      const result = await register(userData)

      expect(mockApi.post).toHaveBeenCalledWith('/auth/register', userData)
      expect(result).toEqual(mockResponse.data)
      expect(loading.value).toBe(false)
      expect(error.value).toBeNull()
    })
  })

  describe('logout', () => {
    it('should logout successfully', async () => {
      const { logout } = useAuth()

      await logout()

      // Vérifier que le token est supprimé et l'utilisateur déconnecté
      expect(localStorage.getItem('token')).toBeNull()
    })
  })
})
```

## 🧪 Tests Frontend - Composants

### Test du composant EventCard

```typescript
// EventCard.test.ts
import { mount } from '@vue/test-utils'
import { describe, it, expect, vi } from 'vitest'
import EventCard from '@/components/EventCard.vue'

describe('EventCard', () => {
  const mockEvent = {
    id: '1',
    title: 'Test Event',
    description: 'Test Description',
    date: '2024-01-01',
    time: '10:00',
    location: {
      address: '123 Test Street',
      coordinates: [48.8566, 2.3522],
    },
    maxParticipants: 10,
    currentParticipants: 5,
    category: 'volunteer',
    image: 'test-image.jpg',
  }

  it('displays event information correctly', () => {
    const wrapper = mount(EventCard, {
      props: { event: mockEvent },
    })

    expect(wrapper.text()).toContain('Test Event')
    expect(wrapper.text()).toContain('Test Description')
    expect(wrapper.text()).toContain('123 Test Street')
    expect(wrapper.text()).toContain('5/10 participants')
  })

  it('emits click event when card is clicked', async () => {
    const wrapper = mount(EventCard, {
      props: { event: mockEvent },
    })

    await wrapper.find('[data-testid="event-card"]').trigger('click')

    expect(wrapper.emitted('click')).toBeTruthy()
    expect(wrapper.emitted('click')[0]).toEqual([mockEvent.id])
  })

  it('shows full capacity indicator when event is full', () => {
    const fullEvent = {
      ...mockEvent,
      currentParticipants: 10,
      maxParticipants: 10,
    }

    const wrapper = mount(EventCard, {
      props: { event: fullEvent },
    })

    expect(wrapper.find('[data-testid="full-capacity"]').exists()).toBe(true)
    expect(wrapper.text()).toContain('Complet')
  })

  it('displays event image when available', () => {
    const wrapper = mount(EventCard, {
      props: { event: mockEvent },
    })

    const image = wrapper.find('[data-testid="event-image"]')
    expect(image.exists()).toBe(true)
    expect(image.attributes('src')).toBe('test-image.jpg')
    expect(image.attributes('alt')).toBe('Test Event')
  })

  it('shows placeholder when no image is available', () => {
    const eventWithoutImage = {
      ...mockEvent,
      image: null,
    }

    const wrapper = mount(EventCard, {
      props: { event: eventWithoutImage },
    })

    expect(wrapper.find('[data-testid="event-image"]').exists()).toBe(false)
    expect(wrapper.find('[data-testid="placeholder-image"]').exists()).toBe(true)
  })

  it('formats date correctly', () => {
    const wrapper = mount(EventCard, {
      props: { event: mockEvent },
    })

    const formattedDate = wrapper.find('[data-testid="event-date"]')
    expect(formattedDate.text()).toContain('1 janvier 2024')
  })

  it('shows distance when location is provided', () => {
    const wrapper = mount(EventCard, {
      props: { event: mockEvent },
      global: {
        provide: {
          userLocation: { lat: 48.8566, lng: 2.3522 },
        },
      },
    })

    expect(wrapper.find('[data-testid="event-distance"]').exists()).toBe(true)
  })
})
```

### Test du composant UserForm

```typescript
// UserForm.test.ts
import { mount } from '@vue/test-utils'
import { describe, it, expect, vi } from 'vitest'
import UserForm from '@/components/UserForm.vue'

describe('UserForm', () => {
  const mockUser = {
    id: '1',
    firstName: 'John',
    lastName: 'Doe',
    email: 'john@example.com',
    phone: '0123456789',
    role: 'volunteer',
  }

  it('renders form fields correctly', () => {
    const wrapper = mount(UserForm, {
      props: { user: mockUser },
    })

    expect(wrapper.find('[data-testid="first-name-input"]').exists()).toBe(true)
    expect(wrapper.find('[data-testid="last-name-input"]').exists()).toBe(true)
    expect(wrapper.find('[data-testid="email-input"]').exists()).toBe(true)
    expect(wrapper.find('[data-testid="phone-input"]').exists()).toBe(true)
  })

  it('populates form with user data', () => {
    const wrapper = mount(UserForm, {
      props: { user: mockUser },
    })

    expect(wrapper.find('[data-testid="first-name-input"]').element.value).toBe('John')
    expect(wrapper.find('[data-testid="last-name-input"]').element.value).toBe('Doe')
    expect(wrapper.find('[data-testid="email-input"]').element.value).toBe('john@example.com')
    expect(wrapper.find('[data-testid="phone-input"]').element.value).toBe('0123456789')
  })

  it('validates required fields', async () => {
    const wrapper = mount(UserForm, {
      props: { user: {} },
    })

    // Clear required fields
    await wrapper.find('[data-testid="first-name-input"]').setValue('')
    await wrapper.find('[data-testid="last-name-input"]').setValue('')
    await wrapper.find('[data-testid="email-input"]').setValue('')

    await wrapper.find('form').trigger('submit')

    expect(wrapper.find('[data-testid="first-name-error"]').exists()).toBe(true)
    expect(wrapper.find('[data-testid="last-name-error"]').exists()).toBe(true)
    expect(wrapper.find('[data-testid="email-error"]').exists()).toBe(true)
  })

  it('validates email format', async () => {
    const wrapper = mount(UserForm, {
      props: { user: mockUser },
    })

    await wrapper.find('[data-testid="email-input"]').setValue('invalid-email')
    await wrapper.find('form').trigger('submit')

    expect(wrapper.find('[data-testid="email-error"]').exists()).toBe(true)
    expect(wrapper.find('[data-testid="email-error"]').text()).toContain('Format d\'email invalide')
  })

  it('emits submit event with form data', async () => {
    const wrapper = mount(UserForm, {
      props: { user: mockUser },
    })

    await wrapper.find('form').trigger('submit')

    expect(wrapper.emitted('submit')).toBeTruthy()
    expect(wrapper.emitted('submit')[0][0]).toEqual({
      firstName: 'John',
      lastName: 'Doe',
      email: 'john@example.com',
      phone: '0123456789',
      role: 'volunteer',
    })
  })

  it('shows loading state during submission', async () => {
    const wrapper = mount(UserForm, {
      props: { user: mockUser, loading: true },
    })

    const submitButton = wrapper.find('[data-testid="submit-button"]')
    expect(submitButton.attributes('disabled')).toBeDefined()
    expect(submitButton.text()).toContain('Enregistrement...')
  })

  it('displays error message when provided', () => {
    const errorMessage = 'Une erreur est survenue'
    const wrapper = mount(UserForm, {
      props: { user: mockUser, error: errorMessage },
    })

    expect(wrapper.find('[data-testid="form-error"]').exists()).toBe(true)
    expect(wrapper.find('[data-testid="form-error"]').text()).toContain(errorMessage)
  })
})
```

## 📊 Métriques de couverture

### Objectifs de couverture

| Type de test | Seuil minimum | Objectif |
|--------------|---------------|----------|
| **Tests unitaires Backend** | 80% | 90% |
| **Tests unitaires Frontend** | 70% | 80% |
| **Tests d'intégration** | 60% | 75% |
| **Tests E2E** | 40% | 60% |

### Rapport de couverture

```bash
# Génération du rapport
npm run test:coverage

# Résultats attendus
----------|---------|----------|---------|---------|-------------------
File      | % Stmts | % Branch | % Funcs | % Lines | Uncovered Line #s
----------|---------|----------|---------|---------|-------------------
All files |   85.71 |    78.57 |   88.89 |   85.71 |
```

## 🚀 Exécution des tests

### Scripts de test

```json
{
  "scripts": {
    "test": "vitest",
    "test:unit": "vitest --run",
    "test:coverage": "vitest --coverage",
    "test:ui": "vitest --ui",
    "test:watch": "vitest --watch"
  }
}
```

### Pipeline CI/CD

```yaml
# .github/workflows/test.yml
name: Tests
on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '20'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run unit tests
        run: npm run test:unit
      
      - name: Generate coverage report
        run: npm run test:coverage
      
      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v3
        with:
          file: ./coverage/lcov.info
```

---

*Ce jeu de tests unitaires garantit la qualité et la fiabilité du code de la plateforme BeneVoclic.* 🚀 