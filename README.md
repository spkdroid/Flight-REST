# Flight-REST

## Credit Application RESTful API with Flight

### Prerequisites
1. PHP 7.2 or higher.
2. Composer for dependency management.

---

### Step 1: Setup the Project

1. **Create a new project directory:**
   ```bash
   mkdir flight-credit-api
   cd flight-credit-api
   ```

2. **Install the Flight framework:**
   ```bash
   composer require mikecao/flight
   ```

3. **Create the project structure:**
   ```bash
   mkdir -p src data
   touch src/index.php data/applications.json
   ```

---

### Step 2: Define the API Endpoints

Edit `src/index.php` and add the following code:

```php
<?php
require '../vendor/autoload.php';

// Helper function to load and save applications
function loadApplications() {
    $file = '../data/applications.json';
    if (!file_exists($file)) {
        file_put_contents($file, json_encode([]));
    }
    return json_decode(file_get_contents($file), true);
}

function saveApplications($applications) {
    file_put_contents('../data/applications.json', json_encode($applications, JSON_PRETTY_PRINT));
}

// Endpoint to submit a credit application
Flight::route('POST /applications', function() {
    $data = Flight::request()->data->getData();
    $applications = loadApplications();

    $newApplication = [
        'id' => uniqid(),
        'name' => $data['name'] ?? 'Unknown',
        'amount' => $data['amount'] ?? 0,
        'status' => 'Pending'
    ];

    $applications[] = $newApplication;
    saveApplications($applications);

    Flight::json(['message' => 'Application submitted', 'application' => $newApplication]);
});

// Endpoint to check the status of a specific application
Flight::route('GET /applications/@id', function($id) {
    $applications = loadApplications();
    $application = array_filter($applications, fn($app) => $app['id'] === $id);

    if ($application) {
        Flight::json(current($application));
    } else {
        Flight::json(['message' => 'Application not found'], 404);
    }
});

// Endpoint to get all credit applications
Flight::route('GET /applications', function() {
    Flight::json(loadApplications());
});

// Endpoint to approve or reject an application
Flight::route('PUT /applications/@id', function($id) {
    $data = Flight::request()->data->getData();
    $applications = loadApplications();
    $found = false;

    foreach ($applications as &$application) {
        if ($application['id'] === $id) {
            $application['status'] = $data['status'] ?? 'Pending';
            $found = true;
            break;
        }
    }

    if ($found) {
        saveApplications($applications);
        Flight::json(['message' => 'Application updated']);
    } else {
        Flight::json(['message' => 'Application not found'], 404);
    }
});

// Start the Flight framework
Flight::start();
```

---

### Step 3: Simulate Data Storage

Create an empty JSON file to store credit applications:
```bash
echo "[]" > data/applications.json
```

---

### Step 4: Start the Server

Run the PHP built-in server to test your API:
```bash
php -S localhost:8000 -t src
```

---

### Step 5: Test the API

Use **cURL** or a tool like **Postman** to interact with the API.

#### Submit a Credit Application
```bash
curl -X POST http://localhost:8000/applications \
-H "Content-Type: application/json" \
-d '{"name": "John Doe", "amount": 5000}'
```

#### Get All Applications
```bash
curl -X GET http://localhost:8000/applications
```

#### Check Application Status
```bash
curl -X GET http://localhost:8000/applications/<application_id>
```

#### Approve or Reject an Application
```bash
curl -X PUT http://localhost:8000/applications/<application_id> \
-H "Content-Type: application/json" \
-d '{"status": "Approved"}'
```

### Step 6: Extend the API
- Add validation for required fields (e.g., `name` and `amount`).
- Implement authentication for sensitive actions like approving/rejecting applications.
- Add pagination to the "Get All Applications" endpoint.

---
