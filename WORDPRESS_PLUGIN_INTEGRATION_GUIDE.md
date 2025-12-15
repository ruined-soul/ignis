# 📚 WordPress Plugin Development & Integration Guide
## Complete Technical Documentation for IndiManga Wallet & Plugin Extensions

---

## Table of Contents

1. [WordPress Plugin Architecture Fundamentals](#1-wordpress-plugin-architecture-fundamentals)
2. [IndiManga Wallet Plugin Deep Dive](#2-indimanga-wallet-plugin-deep-dive)
3. [Inter-Plugin Communication Methods](#3-inter-plugin-communication-methods)
4. [Building Extensions & Additional Features](#4-building-extensions--additional-features)
5. [Firebase Integration Patterns](#5-firebase-integration-patterns)
6. [REST API Development for External Services](#6-rest-api-development-for-external-services)
7. [Security & Best Practices](#7-security--best-practices)
8. [Planning & Architecture for Future Features](#8-planning--architecture-for-future-features)
9. [Complete Code Examples](#9-complete-code-examples)
10. [Troubleshooting & Common Patterns](#10-troubleshooting--common-patterns)

---

## 1. WordPress Plugin Architecture Fundamentals

### 1.1 What is a WordPress Plugin?

A WordPress plugin is a piece of software that extends or modifies WordPress functionality. Plugins are written in PHP and interact with WordPress through:

- **Hooks** (Actions & Filters)
- **WordPress Core Functions**
- **Database Access**
- **REST API**

### 1.2 Plugin File Structure

```
plugin-name/
├── plugin-name.php          # Main plugin file (required)
├── includes/                # Core functionality files
│   ├── class-main.php      # Main plugin class
│   ├── functions.php       # Helper functions
│   └── hooks.php           # Hook handlers
├── admin/                   # Admin panel files
│   ├── settings.php        # Settings page
│   ├── dashboard.php       # Admin dashboard
│   └── css/admin.css       # Admin styles
├── public/                  # Frontend files
│   ├── shortcodes.php      # Shortcode handlers
│   ├── css/style.css       # Frontend styles
│   └── js/script.js        # Frontend JavaScript
├── assets/                  # Static assets
│   ├── images/
│   └── fonts/
├── vendor/                  # Third-party libraries
└── README.md               # Documentation
```

### 1.3 Plugin Header (Required)

Every plugin needs a header comment in the main file:

```php
<?php
/**
 * Plugin Name: Your Plugin Name
 * Description: Brief description of what your plugin does
 * Version: 1.0.0
 * Author: Your Name
 * Author URI: https://yourwebsite.com
 * License: GPL v2 or later
 * Text Domain: your-plugin-slug
 * Domain Path: /languages
 */

if (!defined('ABSPATH')) exit; // Prevent direct access
```

### 1.4 WordPress Hooks: Actions vs Filters

#### Actions
Actions let you **execute code** at specific points in WordPress execution.

```php
// Hook into WordPress action
add_action('init', 'my_function_name');

function my_function_name() {
    // Your code runs when WordPress initializes
}
```

**Common Actions:**
- `init` - WordPress initialization
- `wp_login` - User logs in
- `user_register` - New user registers
- `wp_enqueue_scripts` - Load frontend scripts/styles
- `admin_enqueue_scripts` - Load admin scripts/styles
- `admin_menu` - Add admin menu items
- `save_post` - Post is saved
- `comment_post` - Comment is posted

#### Filters
Filters let you **modify data** before WordPress uses it.

```php
// Hook into WordPress filter
add_filter('the_content', 'modify_content');

function modify_content($content) {
    // Modify and return content
    return $content . '<p>Extra text!</p>';
}
```

**Common Filters:**
- `the_content` - Modify post content
- `the_title` - Modify post title
- `wp_nav_menu_items` - Modify menu items
- `comment_text` - Modify comment text

### 1.5 Creating Custom Hooks

You can create your own hooks for other plugins to use:

```php
// Create custom action
do_action('my_plugin_event', $param1, $param2);

// Create custom filter
$modified_data = apply_filters('my_plugin_filter', $original_data, $context);
```

---

## 2. IndiManga Wallet Plugin Deep Dive

### 2.1 Plugin Overview

**Name:** IndiManga Wallet Pro v2.0.0  
**Purpose:** Firebase-powered digital wallet system with rewards  
**Tech Stack:** PHP, WordPress, Firebase Realtime Database  

### 2.2 Architecture Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                    WordPress Site                            │
│                                                               │
│  ┌────────────────────────────────────────────────────────┐ │
│  │         IndiManga Wallet Plugin                        │ │
│  │                                                          │ │
│  │  ┌──────────────┐  ┌─────────────┐  ┌──────────────┐  │ │
│  │  │  Admin Panel │  │  Shortcodes │  │  Reward      │  │ │
│  │  │  (Settings)  │  │  (Frontend) │  │  System      │  │ │
│  │  └──────────────┘  └─────────────┘  └──────────────┘  │ │
│  │                                                          │ │
│  │  ┌──────────────────────────────────────────────────┐  │ │
│  │  │         Core Wallet Functions                     │  │ │
│  │  │  - Balance Management                             │  │ │
│  │  │  - Transaction Processing                         │  │ │
│  │  │  │  - User Data Management                         │  │ │
│  │  └──────────────────────────────────────────────────┘  │ │
│  │                                                          │ │
│  │  ┌──────────────────────────────────────────────────┐  │ │
│  │  │         Firebase Connection Layer                 │  │ │
│  │  │  - REST API calls (wp_remote_get/request)        │  │ │
│  │  │  - Kreait SDK integration                         │  │ │
│  │  └──────────────────────────────────────────────────┘  │ │
│  └────────────────────────────────────────────────────────┘ │
│                                                               │
└───────────────────────────┬───────────────────────────────────┘
                            │ HTTPS
                            ▼
                  ┌──────────────────┐
                  │   Firebase RTDB   │
                  │  /wallets/        │
                  │  /settings/       │
                  └──────────────────┘
```

### 2.3 Core Components Breakdown

#### 2.3.1 Main Plugin File (`indimanga-wallet.php`)

**Purpose:** Plugin entry point, defines structure

```php
<?php
/**
 * Plugin Name: IndiManga Wallet Pro
 * Version: 2.0.0
 */

// Define constants
define('IMW_PATH', plugin_dir_path(__FILE__));
define('IMW_URL', plugin_dir_url(__FILE__));
define('IMW_VERSION', '2.0.0');

// Include core files
require_once IMW_PATH . 'firebase/connection/init.php';
require_once IMW_PATH . 'firebase/wallet/wallet-functions.php';
require_once IMW_PATH . 'firebase/wallet/wallet-hooks.php';
require_once IMW_PATH . 'firebase/wallet/reward-system.php';

// Register admin menus
add_action('admin_menu', 'imw_register_admin_menus');

// Plugin activation hook
register_activation_hook(__FILE__, 'imw_activate_plugin');
```

**Key Functions:**
- `define()` - Create global constants for paths
- `require_once` - Include necessary files
- `register_activation_hook()` - Run code on plugin activation

#### 2.3.2 Firebase Connection (`firebase/connection/init.php`)

**Purpose:** Establish connection to Firebase

```php
<?php
require_once IMW_PATH . 'firebase/vendor/autoload.php';

use Kreait\Firebase\Factory;

// Initialize Firebase
$firebaseFactory = (new Factory)
    ->withServiceAccount($serviceAccountPath)
    ->withDatabaseUri('https://indimanga-wallet-default-rtdb.firebaseio.com');

$firebaseDB = $firebaseFactory->createDatabase();

// Store as global
global $imw_firebase_db;
$imw_firebase_db = $firebaseDB;
```

**Two Connection Methods:**

**1. Kreait SDK (Recommended for complex operations)**
```php
global $imw_firebase_db;
$reference = $imw_firebase_db->getReference('wallets/' . $user_id);
$data = $reference->getValue();
```

**2. REST API (Simpler, works without SDK)**
```php
$url = 'https://indimanga-wallet-default-rtdb.firebaseio.com/wallets/' . $user_id . '.json';
$response = wp_remote_get($url);
$data = json_decode(wp_remote_retrieve_body($response), true);
```

#### 2.3.3 Wallet Functions (`firebase/wallet/wallet-functions.php`)

**Purpose:** Core CRUD operations for wallet data

**Key Functions:**

```php
// GET: Retrieve balance
function im_wallet_get_balance($user_id) {
    $url = im_get_firebase_db_url() . "/wallets/{$user_id}/balance.json";
    $response = wp_remote_get($url);
    $data = json_decode(wp_remote_retrieve_body($response), true);
    return $data !== null ? floatval($data) : 0;
}

// UPDATE: Set new balance
function im_wallet_update_balance($user_id, $new_balance) {
    $url = im_get_firebase_db_url() . "/wallets/{$user_id}.json";
    $data = [
        'balance' => $new_balance,
        'last_updated' => current_time('mysql')
    ];
    wp_remote_request($url, [
        'method' => 'PATCH',
        'body' => json_encode($data),
        'headers' => ['Content-Type' => 'application/json']
    ]);
}

// TRANSACTION: Credit or Debit
function im_wallet_process_transaction($user_id, $amount, $type, $description = '') {
    $balance = im_wallet_get_balance($user_id);
    
    if ($type === 'debit' && $balance < $amount) {
        return ['success' => false, 'message' => 'Insufficient balance'];
    }
    
    $new_balance = ($type === 'credit') 
        ? $balance + $amount 
        : $balance - $amount;
    
    im_wallet_update_balance($user_id, $new_balance);
    im_wallet_add_transaction($user_id, $type, $amount, $description);
    
    return ['success' => true, 'new_balance' => $new_balance];
}

// LOG: Add transaction record
function im_wallet_add_transaction($user_id, $type, $amount, $description) {
    $txn_id = 'txn_' . time();
    $url = im_get_firebase_db_url() . "/wallets/{$user_id}/transactions/{$txn_id}.json";
    
    $data = [
        'type' => $type,
        'amount' => floatval($amount),
        'description' => sanitize_text_field($description),
        'date' => current_time('mysql')
    ];
    
    wp_remote_request($url, [
        'method' => 'PUT',
        'body' => json_encode($data),
        'headers' => ['Content-Type' => 'application/json']
    ]);
}
```

#### 2.3.4 Reward System (`firebase/wallet/reward-system.php`)

**Purpose:** Automatic reward distribution

**Signup Bonus:**
```php
add_action('user_register', 'imw_award_signup_bonus', 10, 1);

function imw_award_signup_bonus($user_id) {
    $settings = imw_get_settings();
    $signup_bonus = $settings['signup_bonus'] ?? 100;
    
    if ($signup_bonus > 0) {
        imw_initialize_user_wallet($user_id);
        im_wallet_process_transaction(
            $user_id,
            $signup_bonus,
            'credit',
            'Signup Bonus - Welcome!'
        );
        imw_update_user_meta($user_id, 'signup_bonus_claimed', true);
    }
}
```

**Login Rewards:**
```php
add_action('wp_login', 'imw_process_login_rewards', 10, 2);

function imw_process_login_rewards($user_login, $user) {
    $user_id = $user->ID;
    $settings = imw_get_settings();
    $wallet_data = imw_get_user_wallet_data($user_id);
    
    // First login bonus (one-time)
    if (!isset($wallet_data['first_login_bonus_claimed'])) {
        $first_login_bonus = $settings['first_login_bonus'] ?? 50;
        im_wallet_process_transaction($user_id, $first_login_bonus, 'credit', 'First Login Bonus');
        imw_update_user_meta($user_id, 'first_login_bonus_claimed', true);
    }
    
    // Daily login reward
    imw_process_daily_login_reward($user_id, $wallet_data, $settings);
    
    // Weekly streak bonus
    imw_process_weekly_streak_reward($user_id, $wallet_data, $settings);
}
```

**Daily & Streak Logic:**
```php
function imw_process_daily_login_reward($user_id, $wallet_data, $settings) {
    $daily_reward = $settings['daily_login_reward'] ?? 10;
    $last_daily_claim = $wallet_data['last_daily_claim'] ?? '';
    $today = date('Y-m-d');
    
    // Only award once per day
    if ($last_daily_claim !== $today) {
        im_wallet_process_transaction($user_id, $daily_reward, 'credit', 'Daily Login');
        imw_update_user_meta($user_id, 'last_daily_claim', $today);
        
        // Calculate streak
        $yesterday = date('Y-m-d', strtotime('-1 day'));
        $current_streak = $wallet_data['login_streak'] ?? 0;
        
        $new_streak = ($last_daily_claim === $yesterday) 
            ? $current_streak + 1  // Continue streak
            : 1;                    // Reset streak
        
        imw_update_user_meta($user_id, 'login_streak', $new_streak);
    }
}

function imw_process_weekly_streak_reward($user_id, $wallet_data, $settings) {
    $weekly_reward = $settings['weekly_streak_reward'] ?? 100;
    $current_streak = $wallet_data['login_streak'] ?? 0;
    
    // Award bonus every 7 days
    if ($current_streak > 0 && $current_streak % 7 === 0) {
        $streak_milestone = 'week_' . ($current_streak / 7);
        $claimed_milestones = $wallet_data['claimed_milestones'] ?? [];
        
        // Only award once per milestone
        if (!in_array($streak_milestone, $claimed_milestones)) {
            im_wallet_process_transaction(
                $user_id,
                $weekly_reward,
                'credit',
                "Weekly Streak - {$current_streak} Days!"
            );
            $claimed_milestones[] = $streak_milestone;
            imw_update_user_meta($user_id, 'claimed_milestones', $claimed_milestones);
        }
    }
}
```

### 2.4 Firebase Database Structure

```json
{
  "wallets": {
    "123": {  // user_id
      "balance": 350.50,
      "last_updated": "2025-01-15 10:30:00",
      "created_at": "2025-01-01 08:00:00",
      "signup_bonus_claimed": true,
      "first_login_bonus_claimed": true,
      "login_streak": 14,
      "total_logins": 25,
      "last_daily_claim": "2025-01-15",
      "last_weekly_claim": "2025-01-14",
      "claimed_milestones": ["week_1", "week_2"],
      "transactions": {
        "txn_1705312800": {
          "type": "credit",
          "amount": 100,
          "description": "Signup Bonus",
          "date": "2025-01-01 08:00:00"
        },
        "txn_1705399200": {
          "type": "credit",
          "amount": 10,
          "description": "Daily Login",
          "date": "2025-01-02 09:15:00"
        },
        "txn_1705485600": {
          "type": "debit",
          "amount": 50,
          "description": "Purchase: Premium Chapter",
          "date": "2025-01-03 14:20:00"
        }
      }
    }
  },
  "settings": {
    "signup_bonus": 100,
    "first_login_bonus": 50,
    "daily_login_reward": 10,
    "weekly_streak_reward": 100,
    "monthly_streak_reward": 500,
    "currency_name": "Credits",
    "currency_symbol": "💰"
  }
}
```

---

## 3. Inter-Plugin Communication Methods

When building extensions, you need to communicate with the wallet plugin. Here are all available methods:

### 3.1 Method 1: WordPress Hooks (✅ Recommended)

**Best for:** Event-driven integration, loose coupling

#### Creating Hookable Events in Wallet Plugin

**Add to wallet-functions.php:**
```php
function im_wallet_process_transaction($user_id, $amount, $type, $description = '') {
    // ... existing transaction logic ...
    
    // FIRE CUSTOM HOOKS - Other plugins can listen
    if ($result['success']) {
        // After successful transaction
        do_action('imw_transaction_processed', $user_id, $amount, $type, $description, $result);
        
        // Specific event types
        if ($type === 'credit') {
            do_action('imw_credits_added', $user_id, $amount, $description);
        } else {
            do_action('imw_credits_deducted', $user_id, $amount, $description);
        }
        
        // Balance changed
        do_action('imw_balance_updated', $user_id, $result['new_balance'], $result['previous_balance']);
    }
    
    return $result;
}
```

#### Listening to Hooks in Your Extension Plugin

**In your new plugin:**
```php
<?php
/**
 * Plugin Name: IndiManga Features Extended
 * Description: Rating & Store system with wallet integration
 */

// Listen to wallet events
add_action('imw_credits_added', 'myext_on_credits_added', 10, 3);
add_action('imw_credits_deducted', 'myext_on_credits_deducted', 10, 3);
add_action('imw_balance_updated', 'myext_on_balance_changed', 10, 3);

function myext_on_credits_added($user_id, $amount, $description) {
    // React to credit addition
    error_log("User {$user_id} received {$amount} credits for: {$description}");
    
    // Send notification
    myext_send_notification($user_id, "You received {$amount} credits!");
}

function myext_on_credits_deducted($user_id, $amount, $description) {
    // React to purchase
    if (strpos($description, 'Purchase:') !== false) {
        myext_process_purchase_fulfillment($user_id, $description);
    }
}

function myext_on_balance_changed($user_id, $new_balance, $old_balance) {
    // Track balance changes
    myext_log_balance_change($user_id, $old_balance, $new_balance);
}
```

### 3.2 Method 2: Direct Function Calls (✅ Recommended)

**Best for:** Direct data access, simpler than hooks

#### Making Functions Available

**In wallet plugin, ensure functions are declared with `if (!function_exists())`:**
```php
if (!function_exists('im_wallet_get_balance')) {
    function im_wallet_get_balance($user_id) {
        // ... implementation ...
    }
}
```

#### Calling from Your Extension

```php
<?php
// In your extension plugin

function myext_award_rating_reward($user_id, $manga_id, $rating) {
    // CHECK if wallet plugin is active
    if (!function_exists('im_wallet_process_transaction')) {
        error_log('Wallet plugin not active!');
        return false;
    }
    
    // GET current balance
    $balance = im_wallet_get_balance($user_id);
    error_log("User {$user_id} balance: {$balance}");
    
    // AWARD credits for rating
    $reward_amount = 10;
    $result = im_wallet_process_transaction(
        $user_id,
        $reward_amount,
        'credit',
        "Manga Rating Reward - {$manga_id}"
    );
    
    if ($result['success']) {
        return true;
    }
    
    return false;
}

function myext_process_purchase($user_id, $item_id, $price) {
    // CHECK balance first
    if (!function_exists('im_wallet_get_balance')) {
        return ['success' => false, 'message' => 'Wallet unavailable'];
    }
    
    $balance = im_wallet_get_balance($user_id);
    
    if ($balance < $price) {
        return ['success' => false, 'message' => 'Insufficient funds'];
    }
    
    // DEDUCT credits
    $result = im_wallet_process_transaction(
        $user_id,
        $price,
        'debit',
        "Purchase: Item #{$item_id}"
    );
    
    if ($result['success']) {
        // Fulfill purchase
        myext_unlock_item($user_id, $item_id);
        return ['success' => true, 'message' => 'Purchase successful'];
    }
    
    return $result;
}
```

### 3.3 Method 3: Shared Firebase Database (⚠️ Use with Caution)

**Best for:** When you need complete flexibility, but creates tight coupling

```php
<?php
// In your extension plugin

// Get Firebase connection
function myext_get_firebase_connection() {
    // Reuse wallet plugin's connection if available
    global $imw_firebase_db;
    
    if (isset($imw_firebase_db) && $imw_firebase_db) {
        return $imw_firebase_db;
    }
    
    // Or create your own connection
    require_once plugin_dir_path(__FILE__) . 'vendor/autoload.php';
    $firebase = (new Kreait\Firebase\Factory)
        ->withServiceAccount(__DIR__ . '/service-account.json')
        ->withDatabaseUri('https://indimanga-wallet-default-rtdb.firebaseio.com')
        ->createDatabase();
    
    return $firebase;
}

// Direct Firebase access
function myext_direct_balance_access($user_id) {
    $firebase = myext_get_firebase_connection();
    
    // READ
    $balance = $firebase
        ->getReference("wallets/{$user_id}/balance")
        ->getValue();
    
    // WRITE
    $firebase
        ->getReference("wallets/{$user_id}/balance")
        ->set(100.50);
    
    // UPDATE
    $firebase
        ->getReference("wallets/{$user_id}")
        ->update([
            'balance' => 150.75,
            'last_updated' => date('Y-m-d H:i:s')
        ]);
    
    return $balance;
}

// Custom data structure in Firebase
function myext_store_rating_data($user_id, $manga_id, $rating) {
    $firebase = myext_get_firebase_connection();
    
    $rating_data = [
        'user_id' => $user_id,
        'manga_id' => $manga_id,
        'rating' => $rating,
        'timestamp' => time(),
        'date' => date('Y-m-d H:i:s')
    ];
    
    // Store in your own Firebase path
    $firebase
        ->getReference("manga_ratings/{$manga_id}/{$user_id}")
        ->set($rating_data);
}
```

### 3.4 Method 4: WordPress REST API (✅ For External Access)

**Best for:** Telegram bots, mobile apps, external services

#### Creating REST Endpoints

**In wallet plugin or extension:**
```php
<?php
add_action('rest_api_init', 'imw_register_rest_routes');

function imw_register_rest_routes() {
    // GET balance endpoint
    register_rest_route('imw/v1', '/balance/(?P<user_id>\d+)', [
        'methods' => 'GET',
        'callback' => 'imw_rest_get_balance',
        'permission_callback' => 'imw_rest_permission_check',
        'args' => [
            'user_id' => [
                'required' => true,
                'validate_callback' => function($param) {
                    return is_numeric($param);
                }
            ]
        ]
    ]);
    
    // POST transaction endpoint
    register_rest_route('imw/v1', '/transaction', [
        'methods' => 'POST',
        'callback' => 'imw_rest_create_transaction',
        'permission_callback' => 'imw_rest_permission_check'
    ]);
    
    // GET transactions list
    register_rest_route('imw/v1', '/transactions/(?P<user_id>\d+)', [
        'methods' => 'GET',
        'callback' => 'imw_rest_get_transactions',
        'permission_callback' => 'imw_rest_permission_check'
    ]);
}

// Permission check - verify API key
function imw_rest_permission_check($request) {
    $api_key = $request->get_header('X-API-Key');
    $stored_key = get_option('imw_api_key');
    
    if ($api_key === $stored_key) {
        return true;
    }
    
    return new WP_Error(
        'rest_forbidden',
        'Invalid API key',
        ['status' => 403]
    );
}

// Endpoint callback: Get balance
function imw_rest_get_balance($request) {
    $user_id = $request['user_id'];
    
    if (!function_exists('im_wallet_get_balance')) {
        return new WP_Error('wallet_unavailable', 'Wallet plugin not active', ['status' => 503]);
    }
    
    $balance = im_wallet_get_balance($user_id);
    
    return rest_ensure_response([
        'success' => true,
        'user_id' => $user_id,
        'balance' => $balance,
        'currency' => imw_get_settings()['currency_name'] ?? 'Credits'
    ]);
}

// Endpoint callback: Create transaction
function imw_rest_create_transaction($request) {
    $params = $request->get_json_params();
    
    $user_id = absint($params['user_id']);
    $amount = floatval($params['amount']);
    $type = sanitize_text_field($params['type']);
    $description = sanitize_text_field($params['description']);
    
    if (!in_array($type, ['credit', 'debit'])) {
        return new WP_Error('invalid_type', 'Type must be credit or debit', ['status' => 400]);
    }
    
    $result = im_wallet_process_transaction($user_id, $amount, $type, $description);
    
    return rest_ensure_response($result);
}

// Endpoint callback: Get transactions
function imw_rest_get_transactions($request) {
    $user_id = $request['user_id'];
    $limit = $request->get_param('limit') ?? 20;
    
    $transactions = im_wallet_get_transactions($user_id, $limit);
    
    return rest_ensure_response([
        'success' => true,
        'user_id' => $user_id,
        'transactions' => $transactions,
        'count' => count($transactions)
    ]);
}
```

#### Calling REST API from External Service (Python Telegram Bot Example)

```python
import requests

# WordPress REST API configuration
WP_URL = "https://yoursite.com/wp-json"
API_KEY = "your-secret-api-key"

def get_user_balance(user_id):
    """Get wallet balance via REST API"""
    url = f"{WP_URL}/imw/v1/balance/{user_id}"
    headers = {"X-API-Key": API_KEY}
    
    response = requests.get(url, headers=headers)
    
    if response.status_code == 200:
        data = response.json()
        return data['balance']
    else:
        return None

def add_credits(user_id, amount, description):
    """Add credits via REST API"""
    url = f"{WP_URL}/imw/v1/transaction"
    headers = {
        "X-API-Key": API_KEY,
        "Content-Type": "application/json"
    }
    payload = {
        "user_id": user_id,
        "amount": amount,
        "type": "credit",
        "description": description
    }
    
    response = requests.post(url, json=payload, headers=headers)
    return response.json()

# Telegram bot usage
@bot.message_handler(commands=['balance'])
def show_balance(message):
    user_id = get_wp_user_id(message.from_user.id)  # Map Telegram ID to WP user
    balance = get_user_balance(user_id)
    bot.reply_to(message, f"Your balance: {balance} credits")
```

### 3.5 Comparison Table: Which Method to Use?

| Method | Best For | Pros | Cons | Coupling |
|--------|----------|------|------|----------|
| **Hooks** | Event reactions, notifications | Loose coupling, standard WP way | Requires planning hooks | Low ✅ |
| **Function Calls** | Direct data access, simple operations | Easy to use, straightforward | Plugin dependency | Medium ⚠️ |
| **Firebase Direct** | Complex queries, custom data | Complete flexibility | Tight coupling, fragile | High ❌ |
| **REST API** | External services, mobile apps | Platform-independent | More complex setup | None ✅ |

**Recommended Strategy:** Use **Hooks + Function Calls** for WordPress plugins, **REST API** for external services.

---

## 4. Building Extensions & Additional Features

### 4.1 Planning Your Extension Plugin

Before writing code, plan your architecture:

#### Step 1: Define Feature Requirements
```
Feature: Manga Rating System
- Users can rate manga (1-5 stars)
- One rating per user per manga
- Earn 10 credits per rating
- Maximum 5 ratings per day
- Display average rating on manga pages
```

#### Step 2: Identify Integration Points
```
Integration with Wallet Plugin:
- TRIGGER: User submits rating
- ACTION: Award credits via im_wallet_process_transaction()
- HOOK: Listen to imw_credits_added for confirmation
- DATA: Check balance before allowing purchases
```

#### Step 3: Design Database Structure
```
Firebase Structure:
/manga_ratings/
  /{manga_id}/
    /{user_id}/
      rating: 5
      timestamp: 1705312800
      credited: true
    /stats/
      total_ratings: 127
      average_rating: 4.3
      
WordPress Post Meta:
- manga_average_rating
- manga_total_ratings
```

#### Step 4: Create Plugin Structure
```
indimanga-features-extended/
├── indimanga-features.php          # Main file
├── includes/
│   ├── class-rating-system.php    # Rating logic
│   ├── class-store-system.php     # Store logic
│   ├── wallet-integration.php      # Wallet plugin interface
│   └── firebase-connection.php     # Firebase helper
├── admin/
│   ├── settings.php                # Admin settings
│   └── statistics.php              # Rating/store stats
├── public/
│   ├── shortcode-rating.php        # [manga_rating] shortcode
│   ├── shortcode-store.php         # [manga_store] shortcode
│   └── ajax-handlers.php           # AJAX endpoints
├── assets/
│   ├── css/frontend.css
│   └── js/rating.js
└── README.md
```

### 4.2 Complete Example: Manga Rating System Plugin

#### Main Plugin File

```php
<?php
/**
 * Plugin Name: IndiManga Features Extended
 * Description: Rating & Store system with wallet integration
 * Version: 1.0.0
 * Author: YourName
 * Requires Plugins: indimanga-wallet-enhanced
 */

if (!defined('ABSPATH')) exit;

// Plugin constants
define('IMFE_PATH', plugin_dir_path(__FILE__));
define('IMFE_URL', plugin_dir_url(__FILE__));
define('IMFE_VERSION', '1.0.0');

// Check if wallet plugin is active
add_action('admin_init', 'imfe_check_dependencies');
function imfe_check_dependencies() {
    if (!function_exists('im_wallet_get_balance')) {
        add_action('admin_notices', function() {
            echo '<div class="error"><p><strong>IndiManga Features Extended</strong> requires <strong>IndiManga Wallet Pro</strong> to be installed and activated.</p></div>';
        });
        deactivate_plugins(plugin_basename(__FILE__));
    }
}

// Include files
require_once IMFE_PATH . 'includes/wallet-integration.php';
require_once IMFE_PATH . 'includes/class-rating-system.php';
require_once IMFE_PATH . 'includes/firebase-connection.php';
require_once IMFE_PATH . 'public/shortcode-rating.php';
require_once IMFE_PATH . 'public/ajax-handlers.php';
require_once IMFE_PATH . 'admin/settings.php';

// Initialize plugin
add_action('plugins_loaded', 'imfe_init');
function imfe_init() {
    // Initialize rating system
    IMFE_Rating_System::get_instance();
}

// Enqueue assets
add_action('wp_enqueue_scripts', 'imfe_enqueue_assets');
function imfe_enqueue_assets() {
    wp_enqueue_style('imfe-frontend', IMFE_URL . 'assets/css/frontend.css', [], IMFE_VERSION);
    wp_enqueue_script('imfe-rating', IMFE_URL . 'assets/js/rating.js', ['jquery'], IMFE_VERSION, true);
    
    // Pass data to JavaScript
    wp_localize_script('imfe-rating', 'imfeData', [
        'ajaxUrl' => admin_url('admin-ajax.php'),
        'nonce' => wp_create_nonce('imfe_rating_nonce'),
        'userId' => get_current_user_id()
    ]);
}

// Activation hook
register_activation_hook(__FILE__, 'imfe_activate');
function imfe_activate() {
    // Initialize default settings
    if (!get_option('imfe_settings')) {
        $defaults = [
            'rating_reward' => 10,
            'daily_rating_limit' => 5,
            'require_login' => true,
            'enable_reviews' => true
        ];
        update_option('imfe_settings', $defaults);
    }
}
```

#### Wallet Integration Layer

```php
<?php
// includes/wallet-integration.php

if (!defined('ABSPATH')) exit;

/**
 * Wrapper class for wallet plugin integration
 * Provides safe interface with error handling
 */
class IMFE_Wallet_Integration {
    
    /**
     * Check if wallet plugin is available
     */
    public static function is_available() {
        return function_exists('im_wallet_get_balance');
    }
    
    /**
     * Get user balance
     */
    public static function get_balance($user_id) {
        if (!self::is_available()) {
            return 0;
        }
        return im_wallet_get_balance($user_id);
    }
    
    /**
     * Award credits to user
     */
    public static function award_credits($user_id, $amount, $description) {
        if (!self::is_available()) {
            error_log('IMFE: Wallet plugin not available for credit award');
            return ['success' => false, 'message' => 'Wallet unavailable'];
        }
        
        $result = im_wallet_process_transaction(
            $user_id,
            $amount,
            'credit',
            $description
        );
        
        return $result;
    }
    
    /**
     * Deduct credits from user
     */
    public static function deduct_credits($user_id, $amount, $description) {
        if (!self::is_available()) {
            return ['success' => false, 'message' => 'Wallet unavailable'];
        }
        
        // Check balance first
        $balance = self::get_balance($user_id);
        if ($balance < $amount) {
            return ['success' => false, 'message' => 'Insufficient balance'];
        }
        
        $result = im_wallet_process_transaction(
            $user_id,
            $amount,
            'debit',
            $description
        );
        
        return $result;
    }
    
    /**
     * Check if user can afford item
     */
    public static function can_afford($user_id, $price) {
        $balance = self::get_balance($user_id);
        return $balance >= $price;
    }
}
```

#### Rating System Class

```php
<?php
// includes/class-rating-system.php

if (!defined('ABSPATH')) exit;

class IMFE_Rating_System {
    
    private static $instance = null;
    
    public static function get_instance() {
        if (null === self::$instance) {
            self::$instance = new self();
        }
        return self::$instance;
    }
    
    private function __construct() {
        // Hook into WordPress
        add_action('init', [$this, 'init']);
        
        // Listen to wallet events (optional)
        add_action('imw_credits_added', [$this, 'on_credits_added'], 10, 3);
    }
    
    public function init() {
        // Register custom post type for manga (if needed)
        $this->register_manga_post_type();
    }
    
    /**
     * Submit a rating
     */
    public function submit_rating($user_id, $manga_id, $rating, $review_text = '') {
        // Validate
        if (!is_user_logged_in()) {
            return ['success' => false, 'message' => 'Please log in to rate'];
        }
        
        if ($rating < 1 || $rating > 5) {
            return ['success' => false, 'message' => 'Rating must be 1-5 stars'];
        }
        
        // Check if already rated
        if ($this->has_user_rated($user_id, $manga_id)) {
            return ['success' => false, 'message' => 'You have already rated this manga'];
        }
        
        // Check daily limit
        if ($this->has_reached_daily_limit($user_id)) {
            return ['success' => false, 'message' => 'Daily rating limit reached'];
        }
        
        // Store rating in Firebase
        $rating_data = [
            'rating' => intval($rating),
            'review' => sanitize_textarea_field($review_text),
            'timestamp' => time(),
            'date' => current_time('mysql'),
            'credited' => false
        ];
        
        $firebase = $this->get_firebase();
        $firebase
            ->getReference("manga_ratings/{$manga_id}/{$user_id}")
            ->set($rating_data);
        
        // Update manga statistics
        $this->update_manga_stats($manga_id);
        
        // Award credits
        $settings = get_option('imfe_settings');
        $reward = $settings['rating_reward'] ?? 10;
        
        $credit_result = IMFE_Wallet_Integration::award_credits(
            $user_id,
            $reward,
            "Rating Reward - Manga #{$manga_id}"
        );
        
        if ($credit_result['success']) {
            // Mark as credited
            $firebase
                ->getReference("manga_ratings/{$manga_id}/{$user_id}/credited")
                ->set(true);
        }
        
        // Fire custom hook - other plugins can listen
        do_action('imfe_rating_submitted', $user_id, $manga_id, $rating, $reward);
        
        return [
            'success' => true,
            'message' => "Thank you! You earned {$reward} credits!",
            'credits_earned' => $reward,
            'new_balance' => IMFE_Wallet_Integration::get_balance($user_id)
        ];
    }
    
    /**
     * Check if user has rated this manga
     */
    private function has_user_rated($user_id, $manga_id) {
        $firebase = $this->get_firebase();
        $rating = $firebase
            ->getReference("manga_ratings/{$manga_id}/{$user_id}")
            ->getValue();
        
        return !empty($rating);
    }
    
    /**
     * Check daily rating limit
     */
    private function has_reached_daily_limit($user_id) {
        $settings = get_option('imfe_settings');
        $limit = $settings['daily_rating_limit'] ?? 5;
        
        if ($limit == 0) return false; // No limit
        
        $firebase = $this->get_firebase();
        $today = date('Y-m-d');
        
        // Count ratings today
        $user_data = $firebase
            ->getReference("user_ratings/{$user_id}/daily/{$today}")
            ->getValue();
        
        $count = isset($user_data['count']) ? intval($user_data['count']) : 0;
        
        if ($count >= $limit) {
            return true;
        }
        
        // Increment count
        $firebase
            ->getReference("user_ratings/{$user_id}/daily/{$today}/count")
            ->set($count + 1);
        
        return false;
    }
    
    /**
     * Update manga rating statistics
     */
    private function update_manga_stats($manga_id) {
        $firebase = $this->get_firebase();
        
        // Get all ratings for this manga
        $all_ratings = $firebase
            ->getReference("manga_ratings/{$manga_id}")
            ->getValue();
        
        if (empty($all_ratings)) return;
        
        // Calculate average
        $total = 0;
        $count = 0;
        
        foreach ($all_ratings as $user_id => $data) {
            if (isset($data['rating'])) {
                $total += intval($data['rating']);
                $count++;
            }
        }
        
        $average = $count > 0 ? round($total / $count, 2) : 0;
        
        // Store stats
        $stats = [
            'total_ratings' => $count,
            'average_rating' => $average,
            'last_updated' => time()
        ];
        
        $firebase
            ->getReference("manga_stats/{$manga_id}")
            ->set($stats);
        
        // Also update WordPress post meta
        update_post_meta($manga_id, 'manga_average_rating', $average);
        update_post_meta($manga_id, 'manga_total_ratings', $count);
    }
    
    /**
     * Get manga rating stats
     */
    public function get_manga_stats($manga_id) {
        // Try post meta first (cached)
        $average = get_post_meta($manga_id, 'manga_average_rating', true);
        $total = get_post_meta($manga_id, 'manga_total_ratings', true);
        
        if ($average !== '' && $total !== '') {
            return [
                'average_rating' => floatval($average),
                'total_ratings' => intval($total)
            ];
        }
        
        // Fallback to Firebase
        $firebase = $this->get_firebase();
        $stats = $firebase
            ->getReference("manga_stats/{$manga_id}")
            ->getValue();
        
        return $stats ?: ['average_rating' => 0, 'total_ratings' => 0];
    }
    
    /**
     * Get Firebase connection
     */
    private function get_firebase() {
        global $imw_firebase_db;
        
        if (isset($imw_firebase_db) && $imw_firebase_db) {
            return $imw_firebase_db;
        }
        
        // Create own connection if wallet's not available
        require_once IMFE_PATH . 'includes/firebase-connection.php';
        return IMFE_Firebase::get_connection();
    }
    
    /**
     * Register manga custom post type
     */
    private function register_manga_post_type() {
        register_post_type('manga', [
            'labels' => [
                'name' => 'Manga',
                'singular_name' => 'Manga'
            ],
            'public' => true,
            'has_archive' => true,
            'supports' => ['title', 'editor', 'thumbnail', 'custom-fields'],
            'menu_icon' => 'dashicons-book'
        ]);
    }
    
    /**
     * Hook: React when credits are added
     */
    public function on_credits_added($user_id, $amount, $description) {
        // Check if this was a rating reward
        if (strpos($description, 'Rating Reward') !== false) {
            // Send notification, update badges, etc.
            error_log("User {$user_id} earned rating reward: {$amount} credits");
        }
    }
}
```

#### AJAX Handlers

```php
<?php
// public/ajax-handlers.php

if (!defined('ABSPATH')) exit;

// Handle rating submission via AJAX
add_action('wp_ajax_imfe_submit_rating', 'imfe_ajax_submit_rating');
function imfe_ajax_submit_rating() {
    // Verify nonce
    check_ajax_referer('imfe_rating_nonce', 'nonce');
    
    // Get data
    $user_id = get_current_user_id();
    $manga_id = intval($_POST['manga_id']);
    $rating = intval($_POST['rating']);
    $review = sanitize_textarea_field($_POST['review'] ?? '');
    
    // Process rating
    $rating_system = IMFE_Rating_System::get_instance();
    $result = $rating_system->submit_rating($user_id, $manga_id, $rating, $review);
    
    wp_send_json($result);
}

// Get rating stats
add_action('wp_ajax_imfe_get_stats', 'imfe_ajax_get_stats');
add_action('wp_ajax_nopriv_imfe_get_stats', 'imfe_ajax_get_stats');
function imfe_ajax_get_stats() {
    $manga_id = intval($_GET['manga_id']);
    
    $rating_system = IMFE_Rating_System::get_instance();
    $stats = $rating_system->get_manga_stats($manga_id);
    
    wp_send_json_success($stats);
}
```

#### Shortcode

```php
<?php
// public/shortcode-rating.php

if (!defined('ABSPATH')) exit;

// Register shortcode: [manga_rating id="123"]
add_shortcode('manga_rating', 'imfe_rating_shortcode');

function imfe_rating_shortcode($atts) {
    $atts = shortcode_atts([
        'id' => get_the_ID()
    ], $atts);
    
    $manga_id = intval($atts['id']);
    $user_id = get_current_user_id();
    
    // Get stats
    $rating_system = IMFE_Rating_System::get_instance();
    $stats = $rating_system->get_manga_stats($manga_id);
    
    // Get user's rating if exists
    global $imw_firebase_db;
    $user_rating = null;
    if ($user_id && $imw_firebase_db) {
        $user_rating = $imw_firebase_db
            ->getReference("manga_ratings/{$manga_id}/{$user_id}/rating")
            ->getValue();
    }
    
    // Settings
    $settings = get_option('imfe_settings');
    $reward = $settings['rating_reward'] ?? 10;
    
    ob_start();
    ?>
    <div class="imfe-rating-widget" data-manga-id="<?php echo $manga_id; ?>">
        <div class="rating-stats">
            <div class="average-rating">
                <span class="stars"><?php echo str_repeat('⭐', round($stats['average_rating'])); ?></span>
                <span class="number"><?php echo $stats['average_rating']; ?> / 5</span>
            </div>
            <div class="total-ratings">
                <?php echo $stats['total_ratings']; ?> ratings
            </div>
        </div>
        
        <?php if ($user_id): ?>
            <?php if ($user_rating): ?>
                <div class="user-rating">
                    <p>Your rating: <?php echo str_repeat('⭐', $user_rating); ?></p>
                </div>
            <?php else: ?>
                <div class="rating-form">
                    <p class="reward-notice">Rate this manga and earn <strong><?php echo $reward; ?> credits</strong>!</p>
                    <div class="star-selector">
                        <?php for ($i = 1; $i <= 5; $i++): ?>
                            <span class="star" data-rating="<?php echo $i; ?>">☆</span>
                        <?php endfor; ?>
                    </div>
                    <textarea id="rating-review" placeholder="Write a review (optional)"></textarea>
                    <button id="submit-rating" class="button">Submit Rating</button>
                    <div class="rating-message"></div>
                </div>
            <?php endif; ?>
        <?php else: ?>
            <p><a href="<?php echo wp_login_url(get_permalink()); ?>">Log in</a> to rate this manga!</p>
        <?php endif; ?>
    </div>
    
    <style>
        .imfe-rating-widget {
            padding: 20px;
            background: #f9f9f9;
            border-radius: 8px;
            margin: 20px 0;
        }
        .rating-stats {
            text-align: center;
            margin-bottom: 20px;
        }
        .average-rating {
            font-size: 24px;
            margin-bottom: 10px;
        }
        .star-selector {
            font-size: 32px;
            cursor: pointer;
            margin: 15px 0;
        }
        .star-selector .star:hover,
        .star-selector .star.active {
            color: #FFD700;
        }
        #rating-review {
            width: 100%;
            min-height: 80px;
            margin: 10px 0;
            padding: 10px;
        }
        .rating-message {
            margin-top: 10px;
            padding: 10px;
            border-radius: 4px;
        }
        .rating-message.success {
            background: #d4edda;
            color: #155724;
        }
        .rating-message.error {
            background: #f8d7da;
            color: #721c24;
        }
        .reward-notice {
            background: #fff3cd;
            padding: 10px;
            border-radius: 4px;
            margin-bottom: 15px;
        }
    </style>
    <?php
    return ob_get_clean();
}
```

#### Frontend JavaScript

```javascript
// assets/js/rating.js

jQuery(document).ready(function($) {
    
    // Star hover effect
    $('.star-selector .star').on('mouseenter', function() {
        const rating = $(this).data('rating');
        $('.star-selector .star').each(function(index) {
            if (index < rating) {
                $(this).text('★').addClass('active');
            } else {
                $(this).text('☆').removeClass('active');
            }
        });
    });
    
    // Star click - select rating
    let selectedRating = 0;
    $('.star-selector .star').on('click', function() {
        selectedRating = $(this).data('rating');
        $('.star-selector .star').each(function(index) {
            if (index < selectedRating) {
                $(this).text('★').addClass('active');
            } else {
                $(this).text('☆').removeClass('active');
            }
        });
    });
    
    // Submit rating
    $('#submit-rating').on('click', function() {
        const button = $(this);
        const mangaId = $('.imfe-rating-widget').data('manga-id');
        const review = $('#rating-review').val();
        const messageDiv = $('.rating-message');
        
        if (selectedRating === 0) {
            showMessage('Please select a rating', 'error');
            return;
        }
        
        // Disable button
        button.prop('disabled', true).text('Submitting...');
        
        // Submit via AJAX
        $.ajax({
            url: imfeData.ajaxUrl,
            method: 'POST',
            data: {
                action: 'imfe_submit_rating',
                nonce: imfeData.nonce,
                manga_id: mangaId,
                rating: selectedRating,
                review: review
            },
            success: function(response) {
                if (response.success) {
                    showMessage(response.message, 'success');
                    // Hide form after 2 seconds
                    setTimeout(() => {
                        $('.rating-form').slideUp();
                        location.reload(); // Reload to show updated stats
                    }, 2000);
                } else {
                    showMessage(response.message, 'error');
                    button.prop('disabled', false).text('Submit Rating');
                }
            },
            error: function() {
                showMessage('An error occurred. Please try again.', 'error');
                button.prop('disabled', false).text('Submit Rating');
            }
        });
    });
    
    function showMessage(message, type) {
        $('.rating-message')
            .text(message)
            .removeClass('success error')
            .addClass(type)
            .slideDown();
    }
});
```

---

## 5. Firebase Integration Patterns

### 5.1 Firebase Realtime Database Operations

#### Read Operations

```php
// Get single value
$value = $firebase->getReference('path/to/data')->getValue();

// Get with query
$results = $firebase
    ->getReference('wallets')
    ->orderByChild('balance')
    ->limitToFirst(10)
    ->getValue();

// Check if exists
$exists = $firebase->getReference('path')->getSnapshot()->exists();
```

#### Write Operations

```php
// SET - overwrites existing data
$firebase->getReference('wallets/123')->set([
    'balance' => 100,
    'name' => 'User'
]);

// UPDATE - merges with existing data
$firebase->getReference('wallets/123')->update([
    'balance' => 150
]);

// PUSH - creates unique ID
$newRef = $firebase->getReference('transactions')->push([
    'amount' => 50,
    'date' => time()
]);
$newId = $newRef->getKey();

// REMOVE - delete data
$firebase->getReference('wallets/123')->remove();
```

### 5.2 Data Validation & Sanitization

```php
function imfe_validate_rating_data($data) {
    $validated = [];
    
    // Validate rating (1-5)
    $validated['rating'] = max(1, min(5, intval($data['rating'])));
    
    // Sanitize review text
    $validated['review'] = sanitize_textarea_field($data['review']);
    $validated['review'] = wp_kses_post($validated['review']); // Allow some HTML
    
    // Validate user ID
    $validated['user_id'] = absint($data['user_id']);
    
    // Validate manga ID
    $validated['manga_id'] = absint($data['manga_id']);
    
    // Add timestamp
    $validated['timestamp'] = time();
    $validated['date'] = current_time('mysql');
    
    return $validated;
}
```

### 5.3 Error Handling

```php
function imfe_safe_firebase_operation($callback) {
    try {
        return $callback();
    } catch (Kreait\Firebase\Exception\DatabaseException $e) {
        error_log('Firebase Database Error: ' . $e->getMessage());
        return null;
    } catch (Exception $e) {
        error_log('General Firebase Error: ' . $e->getMessage());
        return null;
    }
}

// Usage
$balance = imfe_safe_firebase_operation(function() use ($firebase, $user_id) {
    return $firebase->getReference("wallets/{$user_id}/balance")->getValue();
});
```

---

## 6. REST API Development for External Services

### 6.1 Complete REST API Example

```php
<?php
/**
 * Complete REST API for wallet & features
 */

class IMFE_REST_API {
    
    public function __construct() {
        add_action('rest_api_init', [$this, 'register_routes']);
    }
    
    public function register_routes() {
        $namespace = 'imfe/v1';
        
        // Wallet endpoints
        register_rest_route($namespace, '/balance/(?P<user_id>\d+)', [
            'methods' => 'GET',
            'callback' => [$this, 'get_balance'],
            'permission_callback' => [$this, 'check_permission']
        ]);
        
        register_rest_route($namespace, '/transaction', [
            'methods' => 'POST',
            'callback' => [$this, 'create_transaction'],
            'permission_callback' => [$this, 'check_permission']
        ]);
        
        // Rating endpoints
        register_rest_route($namespace, '/rating', [
            'methods' => 'POST',
            'callback' => [$this, 'submit_rating'],
            'permission_callback' => [$this, 'check_permission']
        ]);
        
        register_rest_route($namespace, '/manga/(?P<manga_id>\d+)/stats', [
            'methods' => 'GET',
            'callback' => [$this, 'get_manga_stats'],
            'permission_callback' => '__return_true' // Public
        ]);
        
        // User stats endpoint
        register_rest_route($namespace, '/user/(?P<user_id>\d+)/stats', [
            'methods' => 'GET',
            'callback' => [$this, 'get_user_stats'],
            'permission_callback' => [$this, 'check_permission']
        ]);
    }
    
    /**
     * Permission check - API key or user authentication
     */
    public function check_permission($request) {
        // Check API key in header
        $api_key = $request->get_header('X-API-Key');
        $stored_key = get_option('imfe_api_key');
        
        if ($api_key && $api_key === $stored_key) {
            return true;
        }
        
        // Check user authentication
        if (is_user_logged_in()) {
            return true;
        }
        
        return new WP_Error(
            'rest_forbidden',
            'Authentication required',
            ['status' => 401]
        );
    }
    
    /**
     * GET /balance/{user_id}
     */
    public function get_balance($request) {
        $user_id = $request['user_id'];
        
        $balance = IMFE_Wallet_Integration::get_balance($user_id);
        $settings = imw_get_settings();
        
        return rest_ensure_response([
            'success' => true,
            'user_id' => $user_id,
            'balance' => $balance,
            'currency' => [
                'name' => $settings['currency_name'] ?? 'Credits',
                'symbol' => $settings['currency_symbol'] ?? '💰'
            ]
        ]);
    }
    
    /**
     * POST /transaction
     * Body: {"user_id": 123, "amount": 50, "type": "credit", "description": "..."}
     */
    public function create_transaction($request) {
        $params = $request->get_json_params();
        
        // Validate
        if (!isset($params['user_id'], $params['amount'], $params['type'])) {
            return new WP_Error('missing_params', 'Required: user_id, amount, type', ['status' => 400]);
        }
        
        $user_id = absint($params['user_id']);
        $amount = floatval($params['amount']);
        $type = sanitize_text_field($params['type']);
        $description = sanitize_text_field($params['description'] ?? 'API Transaction');
        
        if (!in_array($type, ['credit', 'debit'])) {
            return new WP_Error('invalid_type', 'Type must be credit or debit', ['status' => 400]);
        }
        
        // Process
        if ($type === 'credit') {
            $result = IMFE_Wallet_Integration::award_credits($user_id, $amount, $description);
        } else {
            $result = IMFE_Wallet_Integration::deduct_credits($user_id, $amount, $description);
        }
        
        return rest_ensure_response($result);
    }
    
    /**
     * POST /rating
     * Body: {"user_id": 123, "manga_id": 456, "rating": 5, "review": "..."}
     */
    public function submit_rating($request) {
        $params = $request->get_json_params();
        
        $user_id = absint($params['user_id']);
        $manga_id = absint($params['manga_id']);
        $rating = intval($params['rating']);
        $review = sanitize_textarea_field($params['review'] ?? '');
        
        $rating_system = IMFE_Rating_System::get_instance();
        $result = $rating_system->submit_rating($user_id, $manga_id, $rating, $review);
        
        return rest_ensure_response($result);
    }
    
    /**
     * GET /manga/{manga_id}/stats
     */
    public function get_manga_stats($request) {
        $manga_id = $request['manga_id'];
        
        $rating_system = IMFE_Rating_System::get_instance();
        $stats = $rating_system->get_manga_stats($manga_id);
        
        return rest_ensure_response([
            'success' => true,
            'manga_id' => $manga_id,
            'stats' => $stats
        ]);
    }
    
    /**
     * GET /user/{user_id}/stats
     */
    public function get_user_stats($request) {
        $user_id = $request['user_id'];
        
        global $imw_firebase_db;
        
        // Get wallet data
        $wallet_data = imw_get_user_wallet_data($user_id);
        
        // Get rating count
        $user_ratings = $imw_firebase_db
            ->getReference("user_ratings/{$user_id}")
            ->getValue();
        
        $total_ratings = 0;
        if ($user_ratings && isset($user_ratings['daily'])) {
            foreach ($user_ratings['daily'] as $date => $data) {
                $total_ratings += isset($data['count']) ? intval($data['count']) : 0;
            }
        }
        
        return rest_ensure_response([
            'success' => true,
            'user_id' => $user_id,
            'stats' => [
                'balance' => IMFE_Wallet_Integration::get_balance($user_id),
                'login_streak' => $wallet_data['login_streak'] ?? 0,
                'total_logins' => $wallet_data['total_logins'] ?? 0,
                'total_ratings' => $total_ratings
            ]
        ]);
    }
}

// Initialize
new IMFE_REST_API();
```

### 6.2 API Authentication Methods

#### Method 1: API Key (Simple)

```php
// Generate and store API key
function imfe_generate_api_key() {
    $api_key = wp_generate_password(32, false);
    update_option('imfe_api_key', $api_key);
    return $api_key;
}

// Verify API key
function imfe_verify_api_key($key) {
    $stored_key = get_option('imfe_api_key');
    return hash_equals($stored_key, $key);
}
```

#### Method 2: JWT Tokens (Advanced)

```php
require_once 'vendor/autoload.php';
use Firebase\JWT\JWT;

function imfe_generate_jwt($user_id) {
    $secret_key = get_option('imfe_jwt_secret');
    $issued_at = time();
    $expiration = $issued_at + (60 * 60 * 24); // 24 hours
    
    $payload = [
        'iss' => get_site_url(),
        'iat' => $issued_at,
        'exp' => $expiration,
        'user_id' => $user_id
    ];
    
    return JWT::encode($payload, $secret_key, 'HS256');
}

function imfe_verify_jwt($token) {
    try {
        $secret_key = get_option('imfe_jwt_secret');
        $decoded = JWT::decode($token, $secret_key, ['HS256']);
        return $decoded;
    } catch (Exception $e) {
        return false;
    }
}
```

### 6.3 Rate Limiting

```php
function imfe_check_rate_limit($identifier, $limit = 100, $window = 3600) {
    $transient_key = 'rate_limit_' . md5($identifier);
    $requests = get_transient($transient_key);
    
    if ($requests === false) {
        // First request in window
        set_transient($transient_key, 1, $window);
        return true;
    }
    
    if ($requests >= $limit) {
        return false; // Rate limit exceeded
    }
    
    set_transient($transient_key, $requests + 1, $window);
    return true;
}

// Use in permission callback
function imfe_check_permission($request) {
    $api_key = $request->get_header('X-API-Key');
    
    if (!imfe_check_rate_limit($api_key, 1000, 3600)) {
        return new WP_Error('rate_limit', 'Rate limit exceeded', ['status' => 429]);
    }
    
    // ... rest of permission check
}
```

---

## 7. Security & Best Practices

### 7.1 Input Validation & Sanitization

```php
/**
 * Always validate and sanitize user input
 */

// Integers
$user_id = absint($_POST['user_id']); // Always positive

// Floats
$amount = floatval($_POST['amount']);
$amount = max(0, $amount); // Ensure positive

// Strings
$description = sanitize_text_field($_POST['description']);

// Textarea
$review = sanitize_textarea_field($_POST['review']);

// HTML (allow safe tags)
$content = wp_kses_post($_POST['content']);

// URLs
$url = esc_url_raw($_POST['url']);

// Email
$email = sanitize_email($_POST['email']);

// Array
$items = array_map('absint', $_POST['items']);
```

### 7.2 Nonce Verification

```php
// Generate nonce
$nonce = wp_create_nonce('imfe_action_name');

// In form
echo '<input type="hidden" name="imfe_nonce" value="' . wp_create_nonce('imfe_submit_rating') . '">';

// Verify on submission
if (!isset($_POST['imfe_nonce']) || !wp_verify_nonce($_POST['imfe_nonce'], 'imfe_submit_rating')) {
    wp_die('Security check failed');
}

// For AJAX
check_ajax_referer('imfe_rating_nonce', 'nonce');
```

### 7.3 Capability Checks

```php
// Check if user can perform action
if (!current_user_can('manage_options')) {
    wp_die('Unauthorized access');
}

// Custom capabilities
if (!current_user_can('edit_post', $post_id)) {
    return;
}

// Check user ownership
if (get_current_user_id() !== intval($post->post_author)) {
    wp_die('You can only edit your own posts');
}
```

### 7.4 SQL Injection Prevention

```php
global $wpdb;

// WRONG - vulnerable to SQL injection ❌
$results = $wpdb->get_results("SELECT * FROM table WHERE id = {$_GET['id']}");

// CORRECT - use prepared statements ✅
$results = $wpdb->get_results($wpdb->prepare(
    "SELECT * FROM {$wpdb->prefix}table WHERE id = %d",
    $_GET['id']
));

// Multiple parameters
$results = $wpdb->get_results($wpdb->prepare(
    "SELECT * FROM {$wpdb->prefix}table WHERE user_id = %d AND status = %s",
    $user_id,
    $status
));
```

### 7.5 XSS Prevention

```php
// Output escaping
echo esc_html($user_input);           // Plain text
echo esc_attr($user_input);           // HTML attributes
echo esc_url($url);                   // URLs
echo esc_js($javascript_string);      // JavaScript
echo wp_kses_post($html_content);     // Safe HTML tags
```

### 7.6 Firebase Security

```php
/**
 * Firebase security rules (set in Firebase Console)
 */
// rules.json
{
  "rules": {
    "wallets": {
      "$userId": {
        // Users can read their own wallet
        ".read": "auth.uid === $userId",
        // Only server can write (using service account)
        ".write": "auth.uid === 'service-account'"
      }
    },
    "settings": {
      // Anyone can read settings
      ".read": true,
      // Only admins can write
      ".write": "auth.uid === 'admin'"
    }
  }
}
```

```php
// PHP: Always use service account (server-side access)
// Never expose service account credentials to frontend!

// WRONG - exposes credentials ❌
wp_localize_script('script', 'firebase', [
    'apiKey' => 'your-api-key',
    'serviceAccount' => 'credentials.json'
]);

// CORRECT - keep server-side only ✅
$firebase = (new Factory)
    ->withServiceAccount(__DIR__ . '/service-account.json')
    ->createDatabase();
```

### 7.7 Prevent Direct File Access

```php
<?php
// Add to top of every PHP file
if (!defined('ABSPATH')) {
    exit; // Exit if accessed directly
}

// or
defined('ABSPATH') || exit;
```

---

## 8. Planning & Architecture for Future Features

### 8.1 Feature Planning Template

Use this template when planning new features:

```markdown
## Feature: [Feature Name]

### 1. Overview
- **Purpose:** What problem does this solve?
- **Target Users:** Who will use this?
- **Priority:** High / Medium / Low

### 2. Requirements
**Functional:**
- [ ] User can do X
- [ ] System automatically does Y
- [ ] Admin can configure Z

**Non-Functional:**
- [ ] Performance: Load time < 2s
- [ ] Security: All inputs validated
- [ ] Scalability: Handle 10K users

### 3. Technical Design

**Database Structure:**
```
Firebase:
  /feature_data/
    /{user_id}/
      field1: value
      field2: value

WordPress:
  - Post meta: feature_status
  - Options: feature_settings
```

**API Endpoints:**
- GET /api/feature/{id} - Retrieve data
- POST /api/feature - Create new
- PUT /api/feature/{id} - Update
- DELETE /api/feature/{id} - Remove

**Integration Points:**
- Wallet: Award credits on action X
- Ratings: Trigger when rating submitted
- Notifications: Send alert on event Y

### 4. User Flow
1. User navigates to feature page
2. User performs action
3. System validates input
4. Process transaction (if applicable)
5. Update database
6. Show confirmation
7. Award credits/rewards

### 5. Admin Interface
- Settings page: Configure feature options
- Statistics: View usage metrics
- Management: Moderate user content

### 6. Testing Plan
- [ ] Unit tests for core functions
- [ ] Integration tests with wallet
- [ ] UI testing with real users
- [ ] Edge cases handled

### 7. Dependencies
- Required plugins: IndiManga Wallet Pro
- Required libraries: None
- API keys needed: None

### 8. Timeline
- Planning: 2 days
- Development: 5 days
- Testing: 2 days
- Launch: 1 day
```

### 8.2 Architecture Decision Record (ADR)

Document important architectural decisions:

```markdown
# ADR-001: Use Firebase for Real-time Data

## Status
Accepted

## Context
Need to store wallet balances and transactions with real-time updates across multiple devices.

## Decision
Use Firebase Realtime Database instead of WordPress database (MySQL).

## Consequences

**Pros:**
- Real-time synchronization
- Scalable to millions of users
- No need for complex caching
- Offline support

**Cons:**
- Additional service dependency
- Requires Firebase account
- Learning curve for developers
- Costs at scale

## Alternatives Considered
1. WordPress database (MySQL) - rejected due to scaling concerns
2. MongoDB - rejected due to complexity
3. Redis - rejected, better for caching than primary storage
```

### 8.3 Plugin Dependency Management

```php
/**
 * Declare plugin dependencies
 */
class IMFE_Dependencies {
    
    private $required_plugins = [
        'indimanga-wallet-enhanced/indimanga-wallet.php' => 'IndiManga Wallet Pro',
    ];
    
    private $required_functions = [
        'im_wallet_get_balance',
        'im_wallet_process_transaction'
    ];
    
    public function check_dependencies() {
        $missing = [];
        
        // Check if plugins are active
        foreach ($this->required_plugins as $plugin_file => $plugin_name) {
            if (!is_plugin_active($plugin_file)) {
                $missing[] = $plugin_name;
            }
        }
        
        // Check if functions exist
        foreach ($this->required_functions as $function) {
            if (!function_exists($function)) {
                $missing[] = "Function: {$function}";
            }
        }
        
        return $missing;
    }
    
    public function show_dependency_notice() {
        $missing = $this->check_dependencies();
        
        if (!empty($missing)) {
            add_action('admin_notices', function() use ($missing) {
                ?>
                <div class="error">
                    <p><strong>IndiManga Features Extended</strong> requires the following:</p>
                    <ul>
                        <?php foreach ($missing as $item): ?>
                            <li><?php echo esc_html($item); ?></li>
                        <?php endforeach; ?>
                    </ul>
                </div>
                <?php
            });
            
            // Deactivate plugin if dependencies missing
            deactivate_plugins(plugin_basename(__FILE__));
        }
    }
}

$dependencies = new IMFE_Dependencies();
add_action('admin_init', [$dependencies, 'show_dependency_notice']);
```

### 8.4 Versioning & Updates

```php
/**
 * Handle plugin updates
 */
register_activation_hook(__FILE__, 'imfe_activate');
function imfe_activate() {
    $current_version = get_option('imfe_version', '0.0.0');
    
    if (version_compare($current_version, IMFE_VERSION, '<')) {
        // Run upgrade routines
        imfe_upgrade($current_version, IMFE_VERSION);
        update_option('imfe_version', IMFE_VERSION);
    }
}

function imfe_upgrade($from_version, $to_version) {
    // Version-specific upgrades
    if (version_compare($from_version, '1.1.0', '<')) {
        imfe_upgrade_1_1_0();
    }
    
    if (version_compare($from_version, '1.2.0', '<')) {
        imfe_upgrade_1_2_0();
    }
}

function imfe_upgrade_1_1_0() {
    // Add new default settings
    $settings = get_option('imfe_settings');
    $settings['new_feature_enabled'] = true;
    update_option('imfe_settings', $settings);
}
```

---

## 9. Complete Code Examples

### 9.1 Complete Store/Purchase System

```php
<?php
/**
 * Complete Store System Plugin
 * Plugin Name: IndiManga Store
 * Description: Purchase system with wallet integration
 * Version: 1.0.0
 */

if (!defined('ABSPATH')) exit;

// Constants
define('IMS_PATH', plugin_dir_path(__FILE__));
define('IMS_URL', plugin_dir_url(__FILE__));

// Main Store Class
class IndiManga_Store {
    
    private static $instance = null;
    
    public static function get_instance() {
        if (null === self::$instance) {
            self::$instance = new self();
        }
        return self::$instance;
    }
    
    private function __construct() {
        add_action('init', [$this, 'init']);
        add_action('wp_ajax_ims_purchase_item', [$this, 'ajax_purchase_item']);
        add_shortcode('manga_store', [$this, 'store_shortcode']);
    }
    
    public function init() {
        $this->register_post_types();
    }
    
    /**
     * Register store item post type
     */
    private function register_post_types() {
        register_post_type('store_item', [
            'labels' => [
                'name' => 'Store Items',
                'singular_name' => 'Store Item'
            ],
            'public' => true,
            'has_archive' => true,
            'supports' => ['title', 'editor', 'thumbnail'],
            'menu_icon' => 'dashicons-cart',
            'show_in_rest' => true
        ]);
    }
    
    /**
     * Create a store item
     */
    public function create_item($title, $description, $price, $type = 'digital') {
        $post_id = wp_insert_post([
            'post_title' => $title,
            'post_content' => $description,
            'post_type' => 'store_item',
            'post_status' => 'publish'
        ]);
        
        if ($post_id) {
            update_post_meta($post_id, 'item_price', floatval($price));
            update_post_meta($post_id, 'item_type', sanitize_text_field($type));
            update_post_meta($post_id, 'total_purchases', 0);
            
            // Store in Firebase for fast access
            $this->store_item_in_firebase($post_id, $title, $price, $type);
        }
        
        return $post_id;
    }
    
    /**
     * Store item data in Firebase
     */
    private function store_item_in_firebase($item_id, $title, $price, $type) {
        global $imw_firebase_db;
        
        if (!$imw_firebase_db) return;
        
        $imw_firebase_db
            ->getReference("store_items/{$item_id}")
            ->set([
                'title' => $title,
                'price' => $price,
                'type' => $type,
                'created_at' => time()
            ]);
    }
    
    /**
     * Process a purchase
     */
    public function purchase_item($user_id, $item_id) {
        // Get item price
        $price = floatval(get_post_meta($item_id, 'item_price', true));
        
        if ($price <= 0) {
            return ['success' => false, 'message' => 'Invalid item'];
        }
        
        // Check if already owned
        if ($this->user_owns_item($user_id, $item_id)) {
            return ['success' => false, 'message' => 'You already own this item'];
        }
        
        // Check balance
        if (!function_exists('im_wallet_get_balance')) {
            return ['success' => false, 'message' => 'Wallet system unavailable'];
        }
        
        $balance = im_wallet_get_balance($user_id);
        
        if ($balance < $price) {
            return [
                'success' => false,
                'message' => "Insufficient balance. You need {$price} credits.",
                'required' => $price,
                'current' => $balance,
                'deficit' => $price - $balance
            ];
        }
        
        // Process transaction
        $item_title = get_the_title($item_id);
        $transaction = im_wallet_process_transaction(
            $user_id,
            $price,
            'debit',
            "Purchase: {$item_title}"
        );
        
        if (!$transaction['success']) {
            return $transaction;
        }
        
        // Grant item to user
        $this->grant_item_to_user($user_id, $item_id);
        
        // Update statistics
        $total_purchases = intval(get_post_meta($item_id, 'total_purchases', true));
        update_post_meta($item_id, 'total_purchases', $total_purchases + 1);
        
        // Fire hook - other plugins can listen
        do_action('ims_item_purchased', $user_id, $item_id, $price);
        
        return [
            'success' => true,
            'message' => "Purchase successful! {$item_title} is now yours.",
            'new_balance' => $transaction['new_balance'],
            'item_id' => $item_id
        ];
    }
    
    /**
     * Grant item to user
     */
    private function grant_item_to_user($user_id, $item_id) {
        global $imw_firebase_db;
        
        // Store in Firebase
        if ($imw_firebase_db) {
            $imw_firebase_db
                ->getReference("user_purchases/{$user_id}/{$item_id}")
                ->set([
                    'purchased_at' => time(),
                    'date' => current_time('mysql')
                ]);
        }
        
        // Also store in user meta (WordPress)
        $owned_items = get_user_meta($user_id, 'owned_store_items', true);
        if (!is_array($owned_items)) {
            $owned_items = [];
        }
        $owned_items[] = $item_id;
        update_user_meta($user_id, 'owned_store_items', array_unique($owned_items));
        
        // Unlock content if it's a chapter/manga
        $item_type = get_post_meta($item_id, 'item_type', true);
        if ($item_type === 'manga_chapter') {
            $this->unlock_chapter($user_id, $item_id);
        }
    }
    
    /**
     * Check if user owns item
     */
    public function user_owns_item($user_id, $item_id) {
        global $imw_firebase_db;
        
        // Check Firebase first (faster)
        if ($imw_firebase_db) {
            $purchase = $imw_firebase_db
                ->getReference("user_purchases/{$user_id}/{$item_id}")
                ->getValue();
            
            if ($purchase) {
                return true;
            }
        }
        
        // Fallback to WordPress user meta
        $owned_items = get_user_meta($user_id, 'owned_store_items', true);
        return is_array($owned_items) && in_array($item_id, $owned_items);
    }
    
    /**
     * Get user's purchased items
     */
    public function get_user_purchases($user_id) {
        global $imw_firebase_db;
        
        if ($imw_firebase_db) {
            $purchases = $imw_firebase_db
                ->getReference("user_purchases/{$user_id}")
                ->getValue();
            
            return $purchases ?: [];
        }
        
        return get_user_meta($user_id, 'owned_store_items', true) ?: [];
    }
    
    /**
     * Unlock premium chapter
     */
    private function unlock_chapter($user_id, $chapter_id) {
        // Implementation depends on your manga system
        update_user_meta($user_id, "chapter_unlocked_{$chapter_id}", true);
    }
    
    /**
     * AJAX: Purchase item
     */
    public function ajax_purchase_item() {
        check_ajax_referer('ims_purchase_nonce', 'nonce');
        
        $user_id = get_current_user_id();
        $item_id = absint($_POST['item_id']);
        
        if (!$user_id) {
            wp_send_json_error(['message' => 'Please log in to purchase']);
        }
        
        $result = $this->purchase_item($user_id, $item_id);
        
        if ($result['success']) {
            wp_send_json_success($result);
        } else {
            wp_send_json_error($result);
        }
    }
    
    /**
     * Store Shortcode
     */
    public function store_shortcode($atts) {
        $atts = shortcode_atts([
            'category' => 'all',
            'limit' => 12
        ], $atts);
        
        $user_id = get_current_user_id();
        $user_balance = function_exists('im_wallet_get_balance') 
            ? im_wallet_get_balance($user_id) 
            : 0;
        
        // Get store items
        $args = [
            'post_type' => 'store_item',
            'posts_per_page' => intval($atts['limit']),
            'post_status' => 'publish'
        ];
        
        $items = new WP_Query($args);
        
        ob_start();
        ?>
        <div class="ims-store-wrapper">
            <div class="store-header">
                <h2>Store</h2>
                <?php if ($user_id): ?>
                    <div class="user-balance">
                        Your Balance: <strong><?php echo $user_balance; ?> Credits</strong>
                    </div>
                <?php endif; ?>
            </div>
            
            <div class="store-items-grid">
                <?php if ($items->have_posts()): ?>
                    <?php while ($items->have_posts()): $items->the_post(); 
                        $item_id = get_the_ID();
                        $price = get_post_meta($item_id, 'item_price', true);
                        $is_owned = $user_id && $this->user_owns_item($user_id, $item_id);
                    ?>
                        <div class="store-item" data-item-id="<?php echo $item_id; ?>">
                            <?php if (has_post_thumbnail()): ?>
                                <div class="item-image">
                                    <?php the_post_thumbnail('medium'); ?>
                                </div>
                            <?php endif; ?>
                            
                            <div class="item-content">
                                <h3><?php the_title(); ?></h3>
                                <div class="item-excerpt">
                                    <?php echo wp_trim_words(get_the_excerpt(), 15); ?>
                                </div>
                                <div class="item-price">
                                    <?php echo $price; ?> Credits
                                </div>
                                
                                <?php if ($is_owned): ?>
                                    <button class="button owned" disabled>✓ Owned</button>
                                <?php elseif ($user_id): ?>
                                    <button class="button purchase-btn" data-item-id="<?php echo $item_id; ?>" data-price="<?php echo $price; ?>">
                                        Purchase
                                    </button>
                                <?php else: ?>
                                    <a href="<?php echo wp_login_url(get_permalink()); ?>" class="button">
                                        Log in to Purchase
                                    </a>
                                <?php endif; ?>
                            </div>
                        </div>
                    <?php endwhile; ?>
                    <?php wp_reset_postdata(); ?>
                <?php else: ?>
                    <p>No items available yet.</p>
                <?php endif; ?>
            </div>
        </div>
        
        <style>
            .ims-store-wrapper {
                padding: 20px;
            }
            .store-header {
                display: flex;
                justify-content: space-between;
                align-items: center;
                margin-bottom: 30px;
            }
            .user-balance {
                background: #f0f0f0;
                padding: 10px 20px;
                border-radius: 5px;
            }
            .store-items-grid {
                display: grid;
                grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
                gap: 20px;
            }
            .store-item {
                border: 1px solid #ddd;
                border-radius: 8px;
                overflow: hidden;
                transition: transform 0.2s;
            }
            .store-item:hover {
                transform: translateY(-5px);
                box-shadow: 0 4px 12px rgba(0,0,0,0.1);
            }
            .item-image img {
                width: 100%;
                height: 200px;
                object-fit: cover;
            }
            .item-content {
                padding: 15px;
            }
            .item-price {
                font-size: 20px;
                font-weight: bold;
                color: #2271b1;
                margin: 10px 0;
            }
            .button {
                width: 100%;
                padding: 10px;
                background: #2271b1;
                color: white;
                border: none;
                border-radius: 4px;
                cursor: pointer;
            }
            .button.owned {
                background: #46b450;
                cursor: default;
            }
            .button:hover:not(.owned) {
                background: #135e96;
            }
        </style>
        
        <script>
        jQuery(document).ready(function($) {
            $('.purchase-btn').on('click', function() {
                const btn = $(this);
                const itemId = btn.data('item-id');
                const price = btn.data('price');
                
                if (!confirm(`Purchase this item for ${price} credits?`)) {
                    return;
                }
                
                btn.prop('disabled', true).text('Processing...');
                
                $.ajax({
                    url: '<?php echo admin_url('admin-ajax.php'); ?>',
                    method: 'POST',
                    data: {
                        action: 'ims_purchase_item',
                        nonce: '<?php echo wp_create_nonce('ims_purchase_nonce'); ?>',
                        item_id: itemId
                    },
                    success: function(response) {
                        if (response.success) {
                            alert(response.data.message);
                            location.reload(); // Reload to update UI
                        } else {
                            alert(response.data.message);
                            btn.prop('disabled', false).text('Purchase');
                        }
                    },
                    error: function() {
                        alert('An error occurred. Please try again.');
                        btn.prop('disabled', false).text('Purchase');
                    }
                });
            });
        });
        </script>
        <?php
        return ob_get_clean();
    }
}

// Initialize
IndiManga_Store::get_instance();
```

---

## 10. Troubleshooting & Common Patterns

### 10.1 Common Issues & Solutions

#### Issue: Functions not available

```php
// Problem: Calling wallet functions but they don't exist
$balance = im_wallet_get_balance($user_id); // Fatal error

// Solution: Always check if function exists
if (function_exists('im_wallet_get_balance')) {
    $balance = im_wallet_get_balance($user_id);
} else {
    error_log('Wallet plugin not active');
    $balance = 0;
}
```

#### Issue: Firebase connection fails

```php
// Problem: Firebase returns null or errors

// Solution: Add error handling
try {
    global $imw_firebase_db;
    if (!$imw_firebase_db) {
        throw new Exception('Firebase not initialized');
    }
    
    $data = $imw_firebase_db->getReference('path')->getValue();
} catch (Exception $e) {
    error_log('Firebase error: ' . $e->getMessage());
    // Fallback logic
}
```

#### Issue: Hooks not firing

```php
// Problem: Your action hook listener doesn't execute

// Check: Is the hook being fired?
// Add to wallet plugin temporarily:
do_action('imw_transaction_processed', $user_id, $amount, $type);
error_log('Hook fired: imw_transaction_processed');

// Check: Is your listener registered correctly?
add_action('imw_transaction_processed', 'my_function', 10, 3); // Note: 3 parameters
function my_function($user_id, $amount, $type) {
    error_log("Hook received: {$user_id}, {$amount}, {$type}");
}

// Check: Is your plugin loaded after the wallet plugin?
// Solution: Increase priority or use 'plugins_loaded' hook
add_action('plugins_loaded', function() {
    if (function_exists('im_wallet_get_balance')) {
        add_action('imw_transaction_processed', 'my_function', 10, 3);
    }
});
```

### 10.2 Debugging Techniques

```php
/**
 * Enable WordPress debug mode
 * Add to wp-config.php:
 */
define('WP_DEBUG', true);
define('WP_DEBUG_LOG', true);
define('WP_DEBUG_DISPLAY', false);

// Then use error_log() extensively
error_log('Debug: user_id = ' . $user_id);
error_log('Debug: balance = ' . $balance);
error_log('Debug: result = ' . print_r($result, true));

// Check logs at: wp-content/debug.log
```

### 10.3 Performance Optimization

```php
/**
 * Cache expensive operations
 */
function ims_get_item_data($item_id) {
    // Check cache first
    $cache_key = 'item_data_' . $item_id;
    $cached = wp_cache_get($cache_key);
    
    if ($cached !== false) {
        return $cached;
    }
    
    // Fetch from database/Firebase
    $data = expensive_operation($item_id);
    
    // Store in cache for 1 hour
    wp_cache_set($cache_key, $data, '', 3600);
    
    return $data;
}

/**
 * Transients for temporary data
 */
function ims_get_stats() {
    $stats = get_transient('ims_stats');
    
    if ($stats === false) {
        $stats = calculate_stats(); // Expensive operation
        set_transient('ims_stats', $stats, HOUR_IN_SECONDS);
    }
    
    return $stats;
}
```

### 10.4 Testing Patterns

```php
/**
 * Unit testing helper
 */
function ims_test_wallet_integration() {
    $test_user_id = 1; // Admin user
    
    echo "<h3>Wallet Integration Test</h3>";
    
    // Test 1: Check if functions exist
    echo "<p>Function exists: " . (function_exists('im_wallet_get_balance') ? '✓' : '✗') . "</p>";
    
    // Test 2: Get balance
    $balance = im_wallet_get_balance($test_user_id);
    echo "<p>Current balance: {$balance}</p>";
    
    // Test 3: Add credits
    $result = im_wallet_process_transaction($test_user_id, 10, 'credit', 'Test transaction');
    echo "<p>Add credits: " . ($result['success'] ? '✓' : '✗') . " - " . $result['message'] . "</p>";
    
    // Test 4: Deduct credits
    $result = im_wallet_process_transaction($test_user_id, 5, 'debit', 'Test purchase');
    echo "<p>Deduct credits: " . ($result['success'] ? '✓' : '✗') . " - " . $result['message'] . "</p>";
    
    // Test 5: Final balance
    $final_balance = im_wallet_get_balance($test_user_id);
    echo "<p>Final balance: {$final_balance}</p>";
}

// Add admin menu to run tests
add_action('admin_menu', function() {
    add_submenu_page(
        'tools.php',
        'Integration Tests',
        'Integration Tests',
        'manage_options',
        'ims-tests',
        'ims_test_wallet_integration'
    );
});
```

---

## Summary & Quick Reference

### Quick Decision Matrix

**When to use each integration method:**

| Your Need | Use This | Complexity | Coupling |
|-----------|----------|------------|----------|
| React to wallet events | WordPress Hooks | Low | Loose ✅ |
| Get/set wallet data | Direct Function Calls | Low | Medium |
| Complex Firebase queries | Direct Firebase Access | High | Tight ⚠️ |
| External app (Telegram bot) | REST API | Medium | None ✅ |
| Both plugin & external | Hooks + REST API | Medium | Mixed |

### Essential Code Snippets

**Check if wallet plugin is active:**
```php
if (!function_exists('im_wallet_get_balance')) {
    // Wallet plugin not active
}
```

**Award credits:**
```php
im_wallet_process_transaction($user_id, 10, 'credit', 'Rating reward');
```

**Deduct credits:**
```php
im_wallet_process_transaction($user_id, 50, 'debit', 'Purchase: Item');
```

**Listen to wallet events:**
```php
add_action('imw_credits_added', 'my_function', 10, 3);
```

**Create REST endpoint:**
```php
register_rest_route('my/v1', '/endpoint', [
    'methods' => 'GET',
    'callback' => 'my_callback',
    'permission_callback' => 'my_permission_check'
]);
```

### File Structure Template

```
your-plugin/
├── your-plugin.php              # Main file with header
├── includes/
│   ├── class-main.php           # Main plugin class
│   ├── wallet-integration.php   # Wallet interface
│   └── firebase-helper.php      # Firebase functions
├── admin/
│   ├── settings.php             # Settings page
│   └── dashboard.php            # Admin dashboard
├── public/
│   ├── shortcodes.php           # Frontend shortcodes
│   └── ajax-handlers.php        # AJAX endpoints
├── assets/
│   ├── css/
│   └── js/
└── README.md
```

---

## Next Steps

Now that you have this comprehensive guide, you can:

1. **Start small:** Begin with a simple rating system or single feature
2. **Use hooks:** Implement loose coupling via WordPress hooks
3. **Add features incrementally:** Don't try to build everything at once
4. **Test thoroughly:** Use the debugging and testing patterns provided
5. **Document as you go:** Keep track of your architectural decisions

### Additional Resources

- [WordPress Plugin Developer Handbook](https://developer.wordpress.org/plugins/)
- [WordPress Hook Reference](https://developer.wordpress.org/reference/hooks/)
- [Firebase PHP SDK Documentation](https://firebase-php.readthedocs.io/)
- [WordPress REST API Handbook](https://developer.wordpress.org/rest-api/)

---

**Good luck building your IndiManga features! 🚀**

This documentation covers everything from basic WordPress plugin architecture to advanced integration patterns. Use it as your reference guide for planning and implementing future additions to your IndiManga ecosystem.
