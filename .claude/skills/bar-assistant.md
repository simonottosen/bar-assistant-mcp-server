---
name: bar-assistant
description: Search cocktails, get recipes, explore ingredients, and manage your bar using the Bar Assistant API
user_invocable: true
command: bar
---

# Bar Assistant Skill

You have access to a Bar Assistant instance at `http://192.168.1.50:3001/` via its REST API. Use this to search cocktails, retrieve recipes, explore ingredients, and answer any cocktail-related questions.

## API Configuration

- **Base URL**: `http://192.168.1.50:3001`
- **Transport**: SSE (Server-Sent Events) MCP endpoint at `/sse`
- **Auth**: Bearer token via environment variable (already configured in the running instance)

## How to Interact

This Bar Assistant instance is already running as an MCP server. Use the MCP tools directly:

### Available MCP Tools

1. **`smart_search_cocktails`** - Search for cocktails using natural language
   - `query`: Natural language search (e.g., "refreshing gin drinks", "bitter cocktails")
   - `similar_to` / `similar_to_id`: Find cocktails similar to a given drink
   - `ingredient`: Filter by primary ingredient
   - `must_include[]`: Required ingredients
   - `must_exclude[]`: Excluded ingredients
   - `preferred_flavors[]`: Flavor preferences (bitter, sweet, sour, herbal, spicy, fruity, smoky, refreshing, rich, dry)
   - `preferred_strength`: light, medium, or strong
   - `abv_min` / `abv_max`: ABV range
   - `glass_type`: Glass requirement
   - `preparation_method`: shake, stir, build, or muddle
   - `limit`: Results limit (default 20, max 50)

2. **`get_recipe`** - Get detailed cocktail recipes
   - Single: `cocktail_id` or `cocktail_name`
   - Batch: `cocktail_ids[]` or `cocktail_names[]` (5-10x faster than sequential)
   - `include_variations`: Boolean, adds similar cocktails
   - `limit`: Max recipes (default 10, max 20)

3. **`get_ingredient_info`** - Research ingredients
   - `ingredient_name`: The ingredient to look up (e.g., "Campari", "gin", "elderflower liqueur")
   - Returns: description, cocktails using it, substitution suggestions, flavor profile

### Direct API Fallback

If MCP tools are unavailable, you can call the API directly via `curl`:

```bash
# Search cocktails
curl -s "http://192.168.1.50:3001/api/cocktails?filter[name]=negroni&include=ingredients,instructions,tags,glass,method&per_page=10" \
  -H "Authorization: Bearer $BAR_ASSISTANT_TOKEN" \
  -H "Accept: application/json"

# Get cocktail by ID
curl -s "http://192.168.1.50:3001/api/cocktails/{id}?include=ingredients,instructions,tags,glass,method" \
  -H "Authorization: Bearer $BAR_ASSISTANT_TOKEN" \
  -H "Accept: application/json"

# Search by ingredient
curl -s "http://192.168.1.50:3001/api/cocktails?filter[ingredient_name]=campari&include=ingredients,instructions,tags,glass,method&per_page=20" \
  -H "Authorization: Bearer $BAR_ASSISTANT_TOKEN" \
  -H "Accept: application/json"

# List all ingredients
curl -s "http://192.168.1.50:3001/api/ingredients" \
  -H "Authorization: Bearer $BAR_ASSISTANT_TOKEN" \
  -H "Accept: application/json"
```

## Response Formatting

When presenting cocktail results to the user:

- Use the cocktail name as a bold heading
- List ingredients with amounts (already converted to oz from ml by the server)
- Number the instruction steps
- Include ABV, glass type, garnish, and method when available
- Include the direct link to the cocktail in Bar Assistant when available
- For searches, briefly note how many results were found

## Example Interactions

User: "What can I make with mezcal?"
Action: Use `smart_search_cocktails` with `ingredient: "mezcal"` or `query: "mezcal cocktails"`

User: "Show me the recipe for a Paper Plane"
Action: Use `get_recipe` with `cocktail_name: "Paper Plane"`

User: "Give me 5 cocktails similar to an Old Fashioned"
Action: Use `smart_search_cocktails` with `similar_to: "Old Fashioned"` and `limit: 5`

User: "What's a good substitute for Campari?"
Action: Use `get_ingredient_info` with `ingredient_name: "Campari"`

User: "I want something bitter and refreshing, not too strong"
Action: Use `smart_search_cocktails` with `preferred_flavors: ["bitter", "refreshing"]` and `preferred_strength: "light"`

User: "Get me the recipes for Negroni, Manhattan, and Daiquiri"
Action: Use `get_recipe` with `cocktail_names: ["Negroni", "Manhattan", "Daiquiri"]` (batch mode)
