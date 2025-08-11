# PantryPal - Technical Implementation Guide

## Database Schema Design

### Core Tables

#### Users
```sql
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    first_name VARCHAR(100),
    last_name VARCHAR(100),
    dietary_preferences JSONB,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);
```

#### Storage_Locations
```sql
CREATE TABLE storage_locations (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id) ON DELETE CASCADE,
    name VARCHAR(100) NOT NULL,
    type VARCHAR(50) NOT NULL, -- 'fridge', 'pantry', 'freezer', 'custom'
    temperature_range JSONB, -- for smart recommendations
    created_at TIMESTAMP DEFAULT NOW()
);
```

#### Ingredients
```sql
CREATE TABLE ingredients (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(255) NOT NULL,
    category VARCHAR(100), -- 'dairy', 'produce', 'pantry', etc.
    barcode VARCHAR(50) UNIQUE,
    nutritional_info JSONB,
    default_expiration_days INTEGER,
    storage_conditions JSONB,
    created_at TIMESTAMP DEFAULT NOW()
);
```

#### User_Inventory
```sql
CREATE TABLE user_inventory (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id) ON DELETE CASCADE,
    ingredient_id UUID REFERENCES ingredients(id),
    storage_location_id UUID REFERENCES storage_locations(id),
    quantity DECIMAL(10,3) NOT NULL,
    unit VARCHAR(50) NOT NULL,
    expiration_date DATE,
    added_date DATE DEFAULT CURRENT_DATE,
    notes TEXT,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);
```

#### Recipes
```sql
CREATE TABLE recipes (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    title VARCHAR(255) NOT NULL,
    description TEXT,
    instructions JSONB NOT NULL, -- array of steps
    cooking_time INTEGER, -- in minutes
    servings INTEGER,
    difficulty VARCHAR(20), -- 'easy', 'medium', 'hard'
    dietary_tags JSONB, -- ['vegetarian', 'gluten-free']
    cuisine_type VARCHAR(100),
    image_url VARCHAR(500),
    source VARCHAR(255), -- 'user', 'api', 'community'
    user_id UUID REFERENCES users(id), -- null for system recipes
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);
```

#### Recipe_Ingredients
```sql
CREATE TABLE recipe_ingredients (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    recipe_id UUID REFERENCES recipes(id) ON DELETE CASCADE,
    ingredient_id UUID REFERENCES ingredients(id),
    quantity DECIMAL(10,3) NOT NULL,
    unit VARCHAR(50) NOT NULL,
    optional BOOLEAN DEFAULT FALSE,
    substitution_notes TEXT
);
```

#### User_Recipes
```sql
CREATE TABLE user_recipes (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id) ON DELETE CASCADE,
    recipe_id UUID REFERENCES recipes(id) ON DELETE CASCADE,
    status VARCHAR(20) DEFAULT 'saved', -- 'saved', 'favorite', 'cooked'
    rating INTEGER CHECK (rating >= 1 AND rating <= 5),
    notes TEXT,
    cooked_count INTEGER DEFAULT 0,
    last_cooked TIMESTAMP,
    created_at TIMESTAMP DEFAULT NOW()
);
```

#### Shopping_Lists
```sql
CREATE TABLE shopping_lists (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id) ON DELETE CASCADE,
    name VARCHAR(255) DEFAULT 'Shopping List',
    created_at TIMESTAMP DEFAULT NOW()
);
```

#### Shopping_List_Items
```sql
CREATE TABLE shopping_list_items (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    shopping_list_id UUID REFERENCES shopping_lists(id) ON DELETE CASCADE,
    ingredient_id UUID REFERENCES ingredients(id),
    quantity DECIMAL(10,3) NOT NULL,
    unit VARCHAR(50) NOT NULL,
    estimated_price DECIMAL(10,2),
    store VARCHAR(100),
    checked BOOLEAN DEFAULT FALSE,
    added_at TIMESTAMP DEFAULT NOW()
);
```

## API Endpoints Design

### Authentication
```
POST /api/auth/register
POST /api/auth/login
POST /api/auth/refresh
POST /api/auth/logout
GET  /api/auth/profile
PUT  /api/auth/profile
```

### Inventory Management
```
GET    /api/inventory                    # Get user's inventory
POST   /api/inventory                    # Add item to inventory
PUT    /api/inventory/:id                # Update inventory item
DELETE /api/inventory/:id                # Remove item from inventory
POST   /api/inventory/scan               # Scan barcode
POST   /api/inventory/photo              # Photo recognition
GET    /api/inventory/expiring           # Get expiring items
POST   /api/inventory/bulk-import        # Bulk import
```

### Recipe Management
```
GET    /api/recipes                      # Get recipes with filters
GET    /api/recipes/:id                  # Get recipe details
POST   /api/recipes                      # Create custom recipe
PUT    /api/recipes/:id                  # Update recipe
DELETE /api/recipes/:id                  # Delete recipe
GET    /api/recipes/recommendations      # Get recommendations
POST   /api/recipes/:id/save             # Save recipe
POST   /api/recipes/:id/favorite         # Favorite recipe
POST   /api/recipes/:id/cook             # Mark as cooked
```

### Shopping Lists
```
GET    /api/shopping-lists               # Get user's shopping lists
POST   /api/shopping-lists               # Create shopping list
GET    /api/shopping-lists/:id           # Get shopping list details
PUT    /api/shopping-lists/:id           # Update shopping list
DELETE /api/shopping-lists/:id           # Delete shopping list
POST   /api/shopping-lists/:id/items     # Add item to list
PUT    /api/shopping-lists/:id/items/:itemId  # Update list item
DELETE /api/shopping-lists/:id/items/:itemId  # Remove item
POST   /api/shopping-lists/generate      # Generate from recipe
```

### Analytics
```
GET    /api/analytics/waste              # Waste tracking
GET    /api/analytics/usage              # Usage patterns
GET    /api/analytics/costs              # Cost tracking
GET    /api/analytics/nutrition          # Nutritional insights
```

## Mobile App Architecture

### React Native Structure
```
src/
├── components/
│   ├── common/
│   │   ├── Button.tsx
│   │   ├── Input.tsx
│   │   ├── Card.tsx
│   │   └── Modal.tsx
│   ├── inventory/
│   │   ├── InventoryItem.tsx
│   │   ├── AddItemModal.tsx
│   │   └── BarcodeScanner.tsx
│   ├── recipes/
│   │   ├── RecipeCard.tsx
│   │   ├── RecipeDetail.tsx
│   │   └── CookingMode.tsx
│   └── shopping/
│       ├── ShoppingItem.tsx
│       └── ShoppingList.tsx
├── screens/
│   ├── HomeScreen.tsx
│   ├── InventoryScreen.tsx
│   ├── RecipesScreen.tsx
│   ├── ShoppingScreen.tsx
│   └── ProfileScreen.tsx
├── navigation/
│   ├── AppNavigator.tsx
│   ├── TabNavigator.tsx
│   └── StackNavigators.tsx
├── services/
│   ├── api.ts
│   ├── auth.ts
│   ├── inventory.ts
│   ├── recipes.ts
│   └── shopping.ts
├── store/
│   ├── slices/
│   │   ├── authSlice.ts
│   │   ├── inventorySlice.ts
│   │   ├── recipesSlice.ts
│   │   └── shoppingSlice.ts
│   └── store.ts
├── utils/
│   ├── constants.ts
│   ├── helpers.ts
│   └── validation.ts
└── types/
    ├── inventory.ts
    ├── recipes.ts
    └── user.ts
```

### Key Dependencies
```json
{
  "dependencies": {
    "react-native": "0.72.0",
    "@react-navigation/native": "^6.1.0",
    "@react-navigation/bottom-tabs": "^6.5.0",
    "@reduxjs/toolkit": "^1.9.0",
    "react-redux": "^8.1.0",
    "react-native-camera": "^4.2.1",
    "react-native-vision-camera": "^3.0.0",
    "@react-native-async-storage/async-storage": "^1.19.0",
    "react-native-vector-icons": "^10.0.0",
    "react-native-gesture-handler": "^2.12.0",
    "react-native-reanimated": "^3.4.0",
    "react-native-svg": "^13.9.0",
    "react-native-push-notification": "^8.1.1",
    "react-native-image-picker": "^5.6.0",
    "react-native-permissions": "^3.8.0"
  }
}
```

## Web App Architecture

### Next.js Structure
```
src/
├── components/
│   ├── ui/
│   │   ├── Button.tsx
│   │   ├── Input.tsx
│   │   ├── Card.tsx
│   │   └── Modal.tsx
│   ├── layout/
│   │   ├── Header.tsx
│   │   ├── Sidebar.tsx
│   │   └── Footer.tsx
│   ├── dashboard/
│   │   ├── StatsCards.tsx
│   │   ├── RecipeRecommendations.tsx
│   │   └── InventoryOverview.tsx
│   ├── inventory/
│   │   ├── InventoryGrid.tsx
│   │   ├── AddItemForm.tsx
│   │   └── InventoryFilters.tsx
│   └── recipes/
│       ├── RecipeGrid.tsx
│       ├── RecipeDetail.tsx
│       └── RecipeFilters.tsx
├── pages/
│   ├── index.tsx
│   ├── inventory/
│   │   ├── index.tsx
│   │   └── [id].tsx
│   ├── recipes/
│   │   ├── index.tsx
│   │   └── [id].tsx
│   ├── shopping/
│   │   └── index.tsx
│   └── profile/
│       └── index.tsx
├── hooks/
│   ├── useAuth.ts
│   ├── useInventory.ts
│   ├── useRecipes.ts
│   └── useShopping.ts
├── services/
│   ├── api.ts
│   ├── auth.ts
│   └── database.ts
├── utils/
│   ├── constants.ts
│   ├── helpers.ts
│   └── validation.ts
└── types/
    ├── inventory.ts
    ├── recipes.ts
    └── user.ts
```

### Key Dependencies
```json
{
  "dependencies": {
    "next": "^13.4.0",
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "@reduxjs/toolkit": "^1.9.0",
    "react-redux": "^8.1.0",
    "tailwindcss": "^3.3.0",
    "@headlessui/react": "^1.7.0",
    "@heroicons/react": "^2.0.0",
    "recharts": "^2.7.0",
    "react-hook-form": "^7.45.0",
    "zod": "^3.21.0",
    "next-auth": "^4.22.0",
    "prisma": "^4.15.0",
    "@prisma/client": "^4.15.0"
  }
}
```

## Backend Architecture

### Node.js/Express Structure
```
src/
├── controllers/
│   ├── authController.ts
│   ├── inventoryController.ts
│   ├── recipeController.ts
│   └── shoppingController.ts
├── middleware/
│   ├── auth.ts
│   ├── validation.ts
│   ├── errorHandler.ts
│   └── rateLimiter.ts
├── models/
│   ├── User.ts
│   ├── Inventory.ts
│   ├── Recipe.ts
│   └── Shopping.ts
├── routes/
│   ├── auth.ts
│   ├── inventory.ts
│   ├── recipes.ts
│   └── shopping.ts
├── services/
│   ├── barcodeService.ts
│   ├── imageRecognitionService.ts
│   ├── recipeRecommendationService.ts
│   └── notificationService.ts
├── utils/
│   ├── database.ts
│   ├── validation.ts
│   └── helpers.ts
└── app.ts
```

### Key Dependencies
```json
{
  "dependencies": {
    "express": "^4.18.0",
    "cors": "^2.8.5",
    "helmet": "^7.0.0",
    "express-rate-limit": "^6.7.0",
    "bcryptjs": "^2.4.3",
    "jsonwebtoken": "^9.0.0",
    "joi": "^17.9.0",
    "multer": "^1.4.5",
    "sharp": "^0.32.0",
    "node-cron": "^3.0.2",
    "nodemailer": "^6.9.0",
    "firebase-admin": "^11.10.0",
    "openai": "^4.0.0",
    "axios": "^1.4.0"
  }
}
```

## AI/ML Integration

### Photo Recognition Service
```typescript
interface ImageRecognitionResult {
  ingredients: Array<{
    name: string;
    confidence: number;
    boundingBox: {
      x: number;
      y: number;
      width: number;
      height: number;
    };
  }>;
  totalItems: number;
}

class ImageRecognitionService {
  async recognizeIngredients(imageBuffer: Buffer): Promise<ImageRecognitionResult> {
    // Integration with Google Cloud Vision API or similar
    // Custom ML model for food recognition
    // Return structured ingredient data
  }
}
```

### Recipe Recommendation Engine
```typescript
interface RecipeRecommendation {
  recipeId: string;
  score: number;
  matchPercentage: number;
  missingIngredients: string[];
  substitutionSuggestions: Array<{
    original: string;
    substitute: string;
    confidence: number;
  }>;
}

class RecipeRecommendationService {
  async getRecommendations(
    userId: string,
    filters: RecipeFilters
  ): Promise<RecipeRecommendation[]> {
    // Collaborative filtering
    // Content-based filtering
    // Ingredient matching algorithm
    // Return ranked recommendations
  }
}
```

## Security Considerations

### Authentication & Authorization
- JWT tokens with refresh mechanism
- Role-based access control
- API rate limiting
- Input validation and sanitization
- SQL injection prevention

### Data Protection
- End-to-end encryption for sensitive data
- GDPR compliance
- Data anonymization for analytics
- Secure file upload handling
- Regular security audits

### Privacy
- User consent management
- Data retention policies
- Opt-out mechanisms
- Transparent data usage

## Performance Optimization

### Database Optimization
- Proper indexing on frequently queried columns
- Query optimization and caching
- Database connection pooling
- Read replicas for scaling

### Caching Strategy
- Redis for session storage
- CDN for static assets
- API response caching
- Image optimization and caching

### Mobile Optimization
- Offline-first architecture
- Background sync
- Image compression
- Lazy loading
- Progressive web app features

## Deployment & DevOps

### CI/CD Pipeline
```yaml
# .github/workflows/deploy.yml
name: Deploy PantryPal
on:
  push:
    branches: [main]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Run tests
        run: npm test
  deploy:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to production
        run: |
          # Deployment steps
```

### Infrastructure
- **Hosting**: AWS ECS or Google Cloud Run
- **Database**: AWS RDS PostgreSQL
- **File Storage**: AWS S3
- **CDN**: CloudFront
- **Monitoring**: DataDog or New Relic
- **Logging**: ELK Stack

### Environment Configuration
```env
# Production
DATABASE_URL=postgresql://user:pass@host:5432/pantrypal
REDIS_URL=redis://host:6379
JWT_SECRET=your-secret-key
GOOGLE_CLOUD_PROJECT=your-project
AWS_ACCESS_KEY_ID=your-key
AWS_SECRET_ACCESS_KEY=your-secret
```

This technical implementation guide provides a comprehensive foundation for building the PantryPal app with modern best practices, scalability, and security in mind.