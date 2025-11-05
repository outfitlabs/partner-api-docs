# Outfit Travel API - Partner Integration

## What You'll Build

Give your agents a magic button: Click it, get a personalized hotel search link for their client. That's it.

**Your agent clicks "Find Hotels" → You call one endpoint → They get a link → Done.**

The link opens Outfit with hotels pre-matched to their client's preferences. No forms, no configuration, no complexity.

---

## Get Started in 3 Steps

### Step 1: Get your API key

We'll send you an API key. That's all you need to get started.

### Step 2: Call the search endpoint

When your agent wants to find hotels for a client, call this:

```javascript
const response = await fetch('https://api.joinoutfit.com/v1/partner/search', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'X-Outfit-Api-Key': 'your-api-key'
  },
  body: JSON.stringify({
    partner_agent_id: "agent-123",        // Your agent's ID
    partner_client_id: "client-456",        // Your client's ID
    search: {
      query: "Romantic hotels in Paris for anniversary, March 15-20"
    }
  })
});

const result = await response.json();
```

### Step 3: Show them the link

```javascript
if (result.status === 'success') {
  // Give this URL to your agent - they click it and see personalized results for their clients
  const link = result.data.deeplink_url;
  // → https://joinoutfit.com/search/abc123
}
```

**That's it.** Three steps. Your agents can now generate personalized hotel searches for their clients.

---

## What About Errors?

**The API tells you exactly what to do.** First-time setup happens automatically through error messages:

### First time an agent searches:
```javascript
// Error response:
{
  "status": "error",
  "error": {
    "code": "AGENT_NOT_LINKED",
    "message": "Agent must be created before searching.",
    "action_required": "create_agent"
  }
}

// Fix it once (creates/links their account):
await fetch('https://api.joinoutfit.com/v1/partner/create-agent', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'X-Outfit-Api-Key': 'your-api-key'
  },
  body: JSON.stringify({
    partner_agent_id: "agent-123",
    email: "agent123@yourcompany.com",
    first_name: "Jane",
    last_name: "Smith"
  })
});

// Now retry the search - it works!
```

### First time searching for a client:
```javascript
// Error response:
{
  "status": "error",
  "error": {
    "code": "CLIENT_NOT_LINKED",
    "message": "Client must be verified before searching.",
    "action_required": "verify_customer"
  }
}

// Fix it once (creates/links client profile):
await fetch('https://api.joinoutfit.com/v1/partner/verify-customer', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'X-Outfit-Api-Key': 'your-api-key'
  },
  body: JSON.stringify({
    partner_agent_id: "agent-123",
    partner_client_id: "client-456",
    client_info: {
      first_name: "John",
      last_name: "Doe",
      email: "john@example.com"  // Optional
    }
  })
});

// Now retry the search - it works!
```

### After that?
Every search just works. No more errors. ✅

---

## Complete Integration Code

Copy-paste this into your project:

```javascript
/**
 * Generate a personalized hotel search link for a client
 *
 * @param {string} agentId - Your agent's unique ID
 * @param {string} clientId - Your client's unique ID
 * @param {object} searchQuery - What the client is looking for
 * @param {object} agentInfo - Only needed for first-time agent setup
 * @param {object} clientInfo - Only needed for first-time client setup
 * @returns {Promise<string>} - Deeplink URL for the agent to click
 */
async function getHotelSearchLink(agentId, clientId, searchQuery, agentInfo = {}, clientInfo = {}) {
  const search = async () => {
    const response = await fetch('https://api.joinoutfit.com/v1/partner/search', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'X-Outfit-Api-Key': process.env.OUTFIT_API_KEY
      },
      body: JSON.stringify({
        partner_agent_id: agentId,
        partner_client_id: clientId,
        search: { query: searchQuery.text },
        traveler_info: searchQuery.context  // Optional: "Celebrating anniversary, loves boutique hotels"
      })
    });
    return response.json();
  };

  // Try to create the search
  let result = await search();

  // First time agent? Link their account
  if (result.error?.code === 'AGENT_NOT_LINKED') {
    await fetch('https://api.joinoutfit.com/v1/partner/create-agent', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'X-Outfit-Api-Key': process.env.OUTFIT_API_KEY
      },
      body: JSON.stringify({
        partner_agent_id: agentId,
        email: agentInfo.email,
        first_name: agentInfo.firstName,
        last_name: agentInfo.lastName
      })
    });
    result = await search(); // Retry
  }

  // First time client? Link their profile
  if (result.error?.code === 'CLIENT_NOT_LINKED') {
    await fetch('https://api.joinoutfit.com/v1/partner/verify-customer', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'X-Outfit-Api-Key': process.env.OUTFIT_API_KEY
      },
      body: JSON.stringify({
        partner_agent_id: agentId,
        partner_client_id: clientId,
        client_info: {
          first_name: clientInfo.firstName,
          last_name: clientInfo.lastName,
          email: clientInfo.email,
          bio_blurb: clientInfo.preferences  // Optional: "Luxury traveler, prefers boutique hotels"
        }
      })
    });
    result = await search(); // Retry
  }

  if (result.status === 'success') {
    return result.data.deeplink_url;
  }

  throw new Error(result.error?.message || 'Failed to create search');
}

// Usage:
const link = await getHotelSearchLink(
  'agent-123',
  'client-456',
  {
    text: 'Romantic hotels in Paris, March 15-20',
    context: 'Anniversary trip, loves historic neighborhoods'
  },
  { email: 'jane@agency.com', firstName: 'Jane', lastName: 'Smith' },
  { firstName: 'John', lastName: 'Doe', email: 'john@example.com' }
);

// Show link to agent → https://joinoutfit.com/search/abc123
```

**That's your entire integration.** 70 lines of code.

---

## Authentication

**Production:** `https://api.joinoutfit.com`
**Development:** `https://api.outfit-qa.com`

**Header:** `X-Outfit-Api-Key: your-api-key`

We'll provide your API key during onboarding. Keep it secret, keep it safe.

---

## Frequently Asked Questions

<details>
<summary><b>What if I have 50 agents to onboard?</b></summary>

Send us a CSV with their emails and names. We'll link/create all their accounts in 1-2 days and send you a report. Then you can use the API for day-to-day searches.

**CSV Format:**
```csv
partner_agent_id,email,first_name,last_name
agent-001,jane@agency.com,Jane,Smith
agent-002,john@agency.com,John,Doe
```

Email your integration contact to get started.

</details>

<details>
<summary><b>Can I use structured search instead of natural language?</b></summary>

Yes! Instead of `query`, send `criteria`:

```javascript
search: {
  criteria: {
    destination: "Paris, France",
    check_in: "2024-03-15",
    check_out: "2024-03-20",
    guests: 2,
    budget: { amount: 300, currency: "USD" }
  }
}
```

Or send both - `query` takes precedence if both are provided.

</details>

<details>
<summary><b>Do I need the client's email?</b></summary>

Nope! Email is optional. First name and last name are enough.

</details>

<details>
<summary><b>What if there are multiple clients with the same name?</b></summary>

We'll return a `disambiguation_required` response with candidates. Show the list to your agent, they pick the right one, then you call the resolve endpoint. See the [disambiguation section](#handling-disambiguation) for details.

This is rare - we auto-link when there's a clear match.

</details>

<details>
<summary><b>Can I display hotel results in my own UI?</b></summary>

Not yet - Release 1 only returns the deeplink URL.

**Coming in Future roadmap**: We'll add a `properties` array to search results with hotel data (names, prices, ratings, images) so you can build your own UI if you want.

For now, the deeplink gives agents a great experience with personalized results.

</details>

<details>
<summary><b>What about traveler_info?</b></summary>

It's optional but recommended. It helps personalize results:

```javascript
traveler_info: "Celebrating 10th anniversary, loves boutique hotels, first time in Paris"
```

**With it:** Results are hyper-personalized to client preferences
**Without it:** Results still work, just less personalized

We also use it to generate/enrich the client's biography for future searches.

</details>

<details>
<summary><b>How do I test this?</b></summary>

Use the development API: `https://api.outfit-qa.com`

We'll give you a test API key. Create test searches, click the links, see the results.

</details>

---

## API Reference

### POST /v1/partner/search

Generate a personalized hotel search deeplink.

**Request:**
```json
{
  "partner_agent_id": "agent-123",
  "partner_client_id": "client-456",
  "search": {
    "query": "Luxury hotels in Paris, March 15-20"
  },
  "traveler_info": "Anniversary trip, prefers romantic neighborhoods"
}
```

**Response (Success):**
```json
{
  "status": "success",
  "data": {
    "deeplink_url": "https://joinoutfit.com/search/abc123",
    "search_session_id": "abc123",
    "search_results": {
      "query": "Luxury hotels in Paris, March 15-20",
      "criteria": {
        "destination": "Paris, France",
        "check_in": "2024-03-15",
        "check_out": "2024-03-20",
        "guests": 2
      }
    }
  }
}
```

**Response (Error - First Time Agent):**
```json
{
  "status": "error",
  "error": {
    "code": "AGENT_NOT_LINKED",
    "message": "Agent must be created before searching.",
    "action_required": "create_agent"
  }
}
```

**Response (Error - First Time Client):**
```json
{
  "status": "error",
  "error": {
    "code": "CLIENT_NOT_LINKED",
    "message": "Client must be verified before searching.",
    "action_required": "verify_customer"
  }
}
```

<details>
<summary><b>All Error Codes</b></summary>

| Code | When it happens              | What to do |
|------|------------------------------|------------|
| `AGENT_NOT_LINKED` | First time agent             | Call `/v1/partner/create-agent` |
| `CLIENT_NOT_LINKED` | First time client            | Call `/v1/partner/verify-customer` |
| `INVALID_DATES` | Bad check-in/check-out dates | Fix the dates |
| `INVALID_API_KEY` | Wrong/missing API key        | Check your API key |
| `INTERNAL_ERROR` | Server error                 | Retry in a few seconds |

</details>

---

### POST /v1/partner/create-agent

Link an agent's account. Only called when you get `AGENT_NOT_LINKED` error.

**Request:**
```json
{
  "partner_agent_id": "agent-123",
  "email": "agent123@agency.com",
  "first_name": "Jane",
  "last_name": "Smith"
}
```

**Response:**
```json
{
  "status": "success",
  "data": {
    "partner_agent_id": "agent-123",
    "linked": true,
    "existing_account": true
  }
}
```

**What happens:**
- If email exists in Outfit → Links to existing account
- If email doesn't exist → Creates new account and sends password setup email

**After this:** Retry your search request - it will work.

---

### POST /v1/partner/verify-customer

Link a client profile. Only called when you get `CLIENT_NOT_LINKED` error.

**Request:**
```json
{
  "partner_agent_id": "agent-123",
  "partner_client_id": "client-456",
  "client_info": {
    "first_name": "John",
    "last_name": "Doe",
    "email": "john@example.com",
    "bio_blurb": "Luxury traveler, prefers boutique hotels"
  }
}
```

**Response (Success - Auto-Linked):**
```json
{
  "status": "success",
  "data": {
    "partner_client_id": "client-456",
    "linked": true,
    "action": "created",
    "confidence": 1.0
  }
}
```

**What happens:**
- If we find exact match (name + email) → Links to existing profile
- If no match → Creates new profile
- If multiple possible matches → Returns `disambiguation_required` (see below)

**After this:** Retry your search request - it will work.

---

### Handling Disambiguation

Sometimes there are multiple clients with similar names. We'll ask which one is correct.

**Response (Disambiguation Required):**
```json
{
  "status": "disambiguation_required",
  "partner_client_id": "client-456",
  "candidates": [
    {
      "outfit_user_id": "uuid-1",
      "first_name": "John",
      "last_name": "Doe",
      "email": "john@example.com",
      "last_search_at": "2024-03-01T14:30:00Z",
      "match_confidence": 0.85
    },
    {
      "outfit_user_id": "uuid-2",
      "first_name": "John",
      "last_name": "Doe",
      "email": null,
      "last_search_at": null,
      "match_confidence": 0.72
    }
  ]
}
```

**Show these to your agent and ask:** "Which John Doe is this?"

**Then call:**
```javascript
// If they pick option 1:
await fetch('https://api.joinoutfit.com/v1/partner/resolve-customer', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'X-Outfit-Api-Key': 'your-api-key'
  },
  body: JSON.stringify({
    partner_client_id: "client-456",
    action: "link",
    outfit_user_id: "uuid-1"
  })
});

// OR if it's a new client:
await fetch('https://api.joinoutfit.com/v1/partner/resolve-customer', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'X-Outfit-Api-Key': 'your-api-key'
  },
  body: JSON.stringify({
    partner_client_id: "client-456",
    action: "create"
  })
});
```

**After this:** Retry your search request - it will work.

**How often does this happen?** Rarely. We auto-link when there's a clear match (90%+ of the time).

---

## Coming in Release 2

We're adding hotel property data to search results so you can build your own UI if you want:

```json
{
  "status": "success",
  "data": {
    "deeplink_url": "https://joinoutfit.com/search/abc123",
    "search_results": {
      "properties": [
        {
          "property_id": "prop-123",
          "name": "Hotel de Crillon",
          "location": "Place de la Concorde, Paris",
          "rating": 4.8,
          "price_per_night": 450,
          "currency": "USD",
          "image_url": "https://...",
          "fit_score": "95%",
          "fit_text": "Best Romantic getaway",
          "rationale": "Perfect for anniversaries with quiet location...",
          "deeplink": "https://joinoutfit.com/search/abc123?property=prop-123"
        }
      ]
    }
  }
}
```

<details>
<summary><b>Property Fields (Release 2)</b></summary>

- **property_id**: Unique identifier
- **name**: Hotel name
- **location**: Address/neighborhood
- **rating**: 0.0-5.0 star rating
- **price_per_night**: Nightly rate
- **currency**: USD, EUR, etc.
- **image_url**: Primary hotel image
- **fit_score**: Match percentage (string: "95%", "88%")
- **fit_text**: Short label (3-8 words) - display as badge/tag
    - Examples: "Best Romantic getaway", "Perfect for business travel"
- **rationale**: Why this hotel (2-3 sentences) - show in detail view
    - Examples: "This boutique hotel matches their preference for quiet neighborhoods..."
- **deeplink**: Direct link to this specific property (pre-selected in search results)

</details>

**Timeline:** Quick follow after Release 1. We'll email when it's ready.

---

## Need Help?

**Questions?** Email your integration contact
**Issues?** Same - we respond quickly
**Feature requests?** Let us know what you need

We're here to make this easy for you.
