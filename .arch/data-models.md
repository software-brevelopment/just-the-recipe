# Data Models

## Core Recipe Schema

```typescript
interface Recipe {
  // Essential Information
  id: string;                    // Unique identifier
  title: string;                 // Recipe title
  description?: string;          // Brief description
  
  // Source Information
  sourceUrl: string;             // Original URL
  sourceName: string;            // Website name
  author?: string;               // Recipe author
  extractedAt: Date;             // Extraction timestamp
  
  // Recipe Components
  ingredients: Ingredient[];     // Ingredient list
  instructions: Instruction[];   // Step-by-step instructions
  equipment?: Equipment[];       // Required equipment
  
  // Timing Information
  prepTime?: Duration;           // Preparation time
  cookTime?: Duration;           // Cooking time
  totalTime?: Duration;          // Total time
  
  // Servings and Yields
  servings?: {
    amount: number;
    unit?: string;               // "servings", "pieces", "cups", etc.
  };
  yield?: string;                // "Makes 12 cookies", etc.
  
  // Additional Information
  nutrition?: NutritionInfo;     // Nutritional data
  difficulty?: 'easy' | 'medium' | 'hard';
  cuisine?: string;              // Cuisine type
  category?: string;             // Main dish, dessert, etc.
  
  // Media
  images?: RecipeImage[];        // Recipe images
  video?: VideoInfo;             // Video instructions
  
  // Extraction Metadata
  extractionMethod: string;      // How data was extracted
  confidenceScore: number;       // 0-1 confidence in extraction
  missingFields: string[];       // Required fields that couldn't be extracted
}

interface Ingredient {
  id: string;
  text: string;                  // Original text
  amount?: number;               // Numeric amount
  unit?: string;                 // cup, tbsp, oz, etc.
  item: string;                  // Main ingredient name
  notes?: string;                // Additional notes
  processed: boolean;            // Whether AI processing was applied
}

interface Instruction {
  id: string;
  step: number;                  // Step number
  text: string;                  // Instruction text
  temperature?: number;          // Temperature in Fahrenheit
  temperatureUnit?: 'F' | 'C';
  time?: Duration;               // Time for this step
  type?: 'prep' | 'cook' | 'assembly' | 'other';
}

interface Equipment {
  id: string;
  name: string;                  // "Large skillet", "Mixing bowl"
  quantity?: number;             // Number needed
  optional?: boolean;            // Whether equipment is optional
}

interface Duration {
  amount: number;
  unit: 'minutes' | 'hours' | 'days';
}

interface NutritionInfo {
  calories?: number;
  protein?: number;
  carbohydrates?: number;
  fat?: number;
  fiber?: number;
  sodium?: number;
  sugar?: number;
  servingSize?: string;
}

interface RecipeImage {
  url: string;
  alt?: string;
  type: 'main' | 'step' | 'ingredient';
  stepNumber?: number;
}

interface VideoInfo {
  url: string;
  thumbnail?: string;
  duration?: Duration;
  type: 'instructional' | 'promotional';
}
```

## API Response Models

### Success Response
```typescript
interface RecipeExtractionResponse {
  success: true;
  data: Recipe;
  processingTime: number;        // Time in milliseconds
  warnings?: string[];           // Non-critical issues
}
```

### Error Response
```typescript
interface ErrorResponse {
  success: false;
  error: {
    code: string;
    message: string;
    details?: any;
  };
  processingTime: number;
}
```

## Database Schema

### Recipes Table
```sql
CREATE TABLE recipes (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  title TEXT NOT NULL,
  source_url TEXT UNIQUE NOT NULL,
  source_name TEXT NOT NULL,
  extracted_at TIMESTAMP DEFAULT NOW(),
  recipe_data JSONB NOT NULL,
  extraction_method TEXT NOT NULL,
  confidence_score DECIMAL(3,2),
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_recipes_source_url ON recipes(source_url);
CREATE INDEX idx_recipes_confidence ON recipes(confidence_score);
CREATE INDEX idx_recipes_extracted_at ON recipes(extracted_at);
```

### Extraction Logs Table
```sql
CREATE TABLE extraction_logs (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  source_url TEXT NOT NULL,
  extraction_method TEXT NOT NULL,
  success BOOLEAN NOT NULL,
  processing_time INTEGER,
  error_code TEXT,
  error_message TEXT,
  created_at TIMESTAMP DEFAULT NOW()
);
```

## Cache Models

### Redis Cache Structure
```
recipe:cache:{url_hash} -> Recipe (JSON, TTL: 24 hours)
extraction:attempt:{url_hash} -> boolean (TTL: 1 hour)
rate_limit:{ip_address} -> count (TTL: 1 hour)
```
