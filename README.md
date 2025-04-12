# Calendarly: AI-Powered Meeting Scheduler

## Project Overview

Calendarly is an AI-powered meeting scheduling assistant that integrates with Slack and Google Calendar. It uses natural language processing via Google's Gemini API to understand scheduling requests, checks calendar availability through Google MCP servers, and automatically creates meetings with Google Meet links. This document summarizes the complete project including architecture, code structure, pricing strategy, and implementation details.

## Core Concept

The application allows users to schedule meetings directly from Slack by mentioning the Calendarly bot:

```
@Calendarly schedule a 30-minute meeting with @John and @Mary tomorrow at 2pm
```

The system then:
1. Uses Gemini AI to parse the meeting request
2. Checks everyone's availability in Google Calendar
3. Finds an available meeting room (if applicable)
4. Creates the meeting with a Google Meet link
5. Sends a confirmation message with all details

## Technology Stack

- **Backend**: Node.js + Express.js
- **Database**: MongoDB with Mongoose
- **Slack Integration**: Slack Bolt SDK
- **Google Integration**: Google Calendar API, Google Meet
- **AI Processing**: Google Gemini API
- **Cloud Infrastructure**: Google MCP Servers
- **Authentication**: Passport.js
- **Payment Processing**: Stripe
- **Frontend**: EJS templates, Tailwind CSS

## File Structure

```
calendarly-app/
│
├── config/                      # Configuration files
│   ├── passport.js              # Authentication configuration
│   ├── database.js              # MongoDB connection setup
│   └── google-api.js            # Google API configuration
│
├── controllers/                 # Route controllers
│   ├── auth.controller.js       # Authentication controllers
│   ├── billing.controller.js    # Subscription and billing controllers
│   ├── organization.controller.js # Organization management
│   ├── slack.controller.js      # Slack interaction handlers
│   ├── calendar.controller.js   # Calendar operations
│   └── admin.controller.js      # Admin dashboard controllers
│
├── middleware/                  # Custom middleware
│   ├── auth.js                  # Authentication middleware
│   ├── rateLimiter.js           # API rate limiting
│   ├── errorHandler.js          # Global error handling
│   └── logger.js                # Request logging
│
├── models/                      # Database models
│   ├── index.js                 # Model exports
│   ├── organization.model.js    # Organization schema
│   ├── user.model.js            # User schema
│   ├── meeting.model.js         # Meeting schema
│   ├── subscription.model.js    # Subscription schema
│   ├── apiKey.model.js          # API key schema
│   └── usageLog.model.js        # Usage logging schema
│
├── public/                      # Static assets
│   ├── css/                     # Compiled CSS
│   ├── js/                      # Client-side JavaScript
│   └── images/                  # Images and icons
│
├── routes/                      # Express routes
│   ├── api/                     # API routes
│   │   ├── index.js             # API route index
│   │   ├── auth.routes.js       # Authentication endpoints
│   │   ├── organization.routes.js # Organization management
│   │   ├── calendar.routes.js   # Calendar operations
│   │   ├── slack.routes.js      # Slack integration endpoints
│   │   └── admin.routes.js      # Admin-only endpoints
│   │
│   ├── auth.routes.js           # Authentication routes
│   ├── billing.routes.js        # Subscription and payment routes
│   ├── dashboard.routes.js      # Customer dashboard routes
│   └── admin.routes.js          # Admin dashboard routes
│
├── services/                    # Business logic services
│   ├── slack.service.js         # Slack integration service
│   ├── calendar.service.js      # Google Calendar operations
│   ├── subscription.service.js  # Subscription management
│   ├── billing.service.js       # Stripe integration
│   ├── notification.service.js  # Email and in-app notifications
│   └── analytics.service.js     # Usage analytics
│
├── utils/                       # Utility functions
│   ├── dateTime.js              # Date and time helpers
│   ├── formatters.js            # Data formatting helpers
│   ├── validators.js            # Data validation helpers
│   ├── apiResponse.js           # Standardized API responses
│   └── logger.js                # Logging utility
│
├── views/                       # EJS templates
│   ├── partials/                # Reusable template parts
│   │   ├── header.ejs           # Page header
│   │   ├── footer.ejs           # Page footer
│   │   ├── sidebar.ejs          # Dashboard sidebar
│   │   └── alerts.ejs           # Notification alerts
│   │
│   ├── auth/                    # Authentication pages
│   ├── dashboard/               # Customer dashboard
│   ├── admin/                   # Admin dashboard
│   ├── billing/                 # Billing pages
│   ├── landing.ejs              # Marketing landing page
│   └── error.ejs                # Error page
│
├── app.js                       # Express app setup
├── server.js                    # Server entry point
├── package.json                 # Dependencies and scripts
└── .env.example                 # Example environment variables
```

## Key Code Components

### 1. Main Application Files

**package.json**
```json
{
  "name": "calendarly",
  "version": "1.0.0",
  "description": "Meeting scheduler for Slack with Google Calendar integration",
  "main": "server.js",
  "scripts": {
    "start": "node server.js",
    "dev": "nodemon server.js",
    "test": "jest",
    "lint": "eslint ."
  },
  "dependencies": {
    "@slack/bolt": "^3.13.1",
    "bcryptjs": "^2.4.3",
    "connect-mongo": "^5.0.0",
    "dotenv": "^16.3.1",
    "ejs": "^3.1.9",
    "express": "^4.18.2",
    "express-rate-limit": "^6.9.0",
    "express-session": "^1.17.3",
    "googleapis": "^126.0.1",
    "helmet": "^7.0.0",
    "jsonwebtoken": "^9.0.1",
    "mongoose": "^7.4.3",
    "morgan": "^1.10.0",
    "nodemailer": "^6.9.4",
    "passport": "^0.6.0",
    "passport-local": "^1.0.0",
    "stripe": "^13.2.0",
    "winston": "^3.10.0",
    "validator": "^13.11.0"
  },
  "devDependencies": {
    "eslint": "^8.47.0",
    "jest": "^29.6.2",
    "nodemon": "^3.0.1",
    "supertest": "^6.3.3",
    "tailwindcss": "^3.3.3"
  },
  "engines": {
    "node": ">=16.0.0"
  },
  "author": "",
  "license": "UNLICENSED",
  "private": true
}
```

**server.js**
```javascript
// server.js
require('dotenv').config();
const app = require('./app');
const mongoose = require('mongoose');
const logger = require('./utils/logger');

// Get port from environment or default to 3000
const PORT = process.env.PORT || 3000;

// Connect to MongoDB
const MONGODB_URI = process.env.NODE_ENV === 'production' 
  ? process.env.MONGODB_URI_PROD 
  : process.env.MONGODB_URI;

mongoose.connect(MONGODB_URI, {
  useNewUrlParser: true,
  useUnifiedTopology: true
})
.then(() => {
  logger.info('Connected to MongoDB');
  
  // Start the server
  app.listen(PORT, () => {
    logger.info(`Server running on port ${PORT}`);
    logger.info(`Environment: ${process.env.NODE_ENV}`);
  });
})
.catch((error) => {
  logger.error('MongoDB connection error:', error);
  process.exit(1);
});

// Handle unhandled promise rejections
process.on('unhandledRejection', (err) => {
  logger.error('Unhandled Promise Rejection:', err);
  // Don't exit in production, but log the error
  if (process.env.NODE_ENV !== 'production') {
    process.exit(1);
  }
});
```

**app.js**
```javascript
// app.js
const express = require('express');
const session = require('express-session');
const MongoStore = require('connect-mongo');
const passport = require('passport');
const helmet = require('helmet');
const morgan = require('morgan');
const path = require('path');
const rateLimit = require('express-rate-limit');
const { App, ExpressReceiver } = require('@slack/bolt');
const { errorHandler } = require('./middleware/errorHandler');
const logger = require('./utils/logger');

// Import routes
const authRoutes = require('./routes/auth.routes');
const billingRoutes = require('./routes/billing.routes');
const dashboardRoutes = require('./routes/dashboard.routes');
const adminRoutes = require('./routes/admin.routes');
const apiRoutes = require('./routes/api');

// Create Express app
const app = express();

// Security headers
app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      scriptSrc: ["'self'", 'https://js.stripe.com'],
      frameSrc: ["'self'", 'https://js.stripe.com', 'https://hooks.slack.com'],
      connectSrc: ["'self'", 'https://api.stripe.com'],
      imgSrc: ["'self'", 'data:', 'https://secure.gravatar.com', 'https://avatar.slack-edge.com'],
      styleSrc: ["'self'", "'unsafe-inline'", 'https://fonts.googleapis.com'],
      fontSrc: ["'self'", 'https://fonts.gstatic.com']
    }
  }
}));

// Set up view engine
app.set('view engine', 'ejs');
app.set('views', path.join(__dirname, 'views'));

// Middleware
if (process.env.NODE_ENV === 'development') {
  app.use(morgan('dev'));
} else {
  app.use(morgan('combined', { stream: { write: message => logger.info(message.trim()) } }));
}

// Body parser middleware
app.use(express.json());
app.use(express.urlencoded({ extended: false }));

// Serve static files
app.use(express.static(path.join(__dirname, 'public')));

// Special handling for Stripe webhook route
app.use('/billing/webhook', express.raw({ type: 'application/json' }), (req, res, next) => {
  req.rawBody = req.body;
  next();
});

// Session configuration
app.use(session({
  secret: process.env.SESSION_SECRET,
  resave: false,
  saveUninitialized: false,
  store: MongoStore.create({
    mongoUrl: process.env.NODE_ENV === 'production' 
      ? process.env.MONGODB_URI_PROD 
      : process.env.MONGODB_URI,
    collectionName: 'sessions'
  }),
  cookie: {
    secure: process.env.NODE_ENV === 'production',
    httpOnly: true,
    maxAge: 7 * 24 * 60 * 60 * 1000 // 7 days
  }
}));

// Initialize Passport
require('./config/passport');
app.use(passport.initialize());
app.use(passport.session());

// Rate limiting
const apiLimiter = rateLimit({
  windowMs: process.env.RATE_LIMIT_WINDOW_MS || 15 * 60 * 1000, // 15 minutes by default
  max: process.env.RATE_LIMIT_MAX_REQUESTS || 100, // Limit each IP to 100 requests per windowMs
  standardHeaders: true,
  legacyHeaders: false,
  message: 'Too many requests from this IP, please try again later'
});

app.use('/api', apiLimiter);

// Configure Slack Bolt
const receiver = new ExpressReceiver({
  signingSecret: process.env.SLACK_SIGNING_SECRET,
  endpoints: '/slack/events',
  processBeforeResponse: true
});

const slackApp = new App({
  receiver,
  token: process.env.SLACK_BOT_TOKEN
});

// Initialize Slack event handlers
require('./services/slack.service')(slackApp);

// Add receiver's routes to Express
app.use(receiver.router);

// Pass user to all templates
app.use((req, res, next) => {
  res.locals.user = req.user || null;
  res.locals.isAuthenticated = req.isAuthenticated();
  next();
});

// Routes
app.use('/', authRoutes);
app.use('/billing', billingRoutes);
app.use('/dashboard', dashboardRoutes);
app.use('/admin', adminRoutes);
app.use('/api', apiRoutes);

// Home route
app.get('/', (req, res) => {
  res.render('landing');
});

// Handle 404
app.use((req, res, next) => {
  res.status(404).render('error', { 
    title: '404 - Page Not Found',
    message: 'The page you are looking for does not exist.'
  });
});

// Error handler middleware
app.use(errorHandler);

module.exports = app;
```

### 2. Database Models

**models/index.js**
```javascript
// models/index.js - Database models for multi-tenant SaaS
const mongoose = require('mongoose');
const Schema = mongoose.Schema;

// Customer/Organization Schema
const OrganizationSchema = new Schema({
  name: { type: String, required: true },
  domain: { type: String, required: true, unique: true },
  createdAt: { type: Date, default: Date.now },
  active: { type: Boolean, default: true },
  plan: { 
    type: String, 
    enum: ['free', 'basic', 'pro', 'enterprise'], 
    default: 'free' 
  },
  slackTeamId: { type: String, unique: true },
  slackBotToken: { type: String },
  slackSigningSecret: { type: String },
  googleRefreshToken: { type: String },
  meetingRooms: [{
    email: String,
    name: String,
    capacity: Number,
    location: String
  }],
  usageStats: {
    totalMeetingsScheduled: { type: Number, default: 0 },
    lastMeetingScheduled: Date
  },
  usageLimits: {
    maxMeetingsPerMonth: { type: Number, default: 10 }, // Free tier limit
    maxAttendeesPerMeeting: { type: Number, default: 5 } // Free tier limit
  }
});

// User Schema
const UserSchema = new Schema({
  email: { type: String, required: true, unique: true },
  password: { type: String, required: true },
  name: { type: String, required: true },
  role: { 
    type: String, 
    enum: ['admin', 'manager', 'user'], 
    default: 'user' 
  },
  organizationId: { 
    type: mongoose.Schema.Types.ObjectId, 
    ref: 'Organization',
    required: true
  },
  createdAt: { type: Date, default: Date.now },
  lastLogin: Date
});

// Meeting Schema (for analytics and history)
const MeetingSchema = new Schema({
  organizationId: { 
    type: mongoose.Schema.Types.ObjectId, 
    ref: 'Organization',
    required: true
  },
  summary: String,
  description: String,
  startTime: { type: Date, required: true },
  endTime: { type: Date, required: true },
  attendees: [String],
  googleEventId: String,
  meetLink: String,
  roomName: String,
  createdBy: String,
  createdAt: { type: Date, default: Date.now }
});

// Subscription/Billing Schema
const SubscriptionSchema = new Schema({
  organizationId: { 
    type: mongoose.Schema.Types.ObjectId, 
    ref: 'Organization',
    required: true
  },
  plan: { 
    type: String, 
    enum: ['free', 'basic', 'pro', 'enterprise'], 
    required: true 
  },
  status: { 
    type: String, 
    enum: ['active', 'past_due', 'canceled', 'trialing'], 
    default: 'active' 
  },
  startDate: { type: Date, default: Date.now },
  endDate: Date,
  billingCycle: { 
    type: String, 
    enum: ['monthly', 'yearly'], 
    default: 'monthly' 
  },
  price: Number,
  paymentMethod: {
    type: { type: String },
    last4: String,
    expiryMonth: Number,
    expiryYear: Number
  },
  billingHistory: [{
    date: Date,
    amount: Number,
    status: { 
      type: String, 
      enum: ['paid', 'failed', 'pending']
    }
  }]
});

// API Key Schema (for external integrations)
const ApiKeySchema = new Schema({
  organizationId: { 
    type: mongoose.Schema.Types.ObjectId, 
    ref: 'Organization',
    required: true
  },
  key: { type: String, required: true },
  name: { type: String, required: true },
  createdAt: { type: Date, default: Date.now },
  lastUsed: Date,
  expiresAt: Date,
  active: { type: Boolean, default: true }
});

// Usage Log Schema (for tracking and analytics)
const UsageLogSchema = new Schema({
  organizationId: { 
    type: mongoose.Schema.Types.ObjectId, 
    ref: 'Organization',
    required: true
  },
  action: { type: String, required: true },
  timestamp: { type: Date, default: Date.now },
  details: Schema.Types.Mixed,
  userId: String,
  ipAddress: String
});

const Organization = mongoose.model('Organization', OrganizationSchema);
const User = mongoose.model('User', UserSchema);
const Meeting = mongoose.model('Meeting', MeetingSchema);
const Subscription = mongoose.model('Subscription', SubscriptionSchema);
const ApiKey = mongoose.model('ApiKey', ApiKeySchema);
const UsageLog = mongoose.model('UsageLog', UsageLogSchema);

module.exports = {
  Organization,
  User,
  Meeting,
  Subscription,
  ApiKey,
  UsageLog
};
```

### 3. Services

**services/subscription.service.js**
```javascript
// services/subscription.js - Manage subscriptions and enforce usage limits
const { Organization, Subscription, Meeting, UsageLog } = require('../models');

/**
 * Service to manage subscription plans and enforce usage limits
 */
class SubscriptionService {
  /**
   * Plan definitions with features and limits
   */
  static PLANS = {
    free: {
      name: 'Free',
      price: 0,
      features: ['Basic scheduling', 'Google Calendar integration', 'Google Meet links'],
      limits: {
        maxMeetingsPerMonth: 10,
        maxAttendeesPerMeeting: 5,
        roomBooking: false,
        recurringMeetings: false,
        customBranding: false,
        analytics: false,
        prioritySupport: false
      }
    },
    basic: {
      name: 'Basic',
      price: 9.99,
      features: [
        'Everything in Free',
        'Up to 50 meetings per month',
        'Up to 15 attendees per meeting',
        'Room booking'
      ],
      limits: {
        maxMeetingsPerMonth: 50,
        maxAttendeesPerMeeting: 15,
        roomBooking: true,
        recurringMeetings: false,
        customBranding: false,
        analytics: false,
        prioritySupport: false
      }
    },
    pro: {
      name: 'Professional',
      price: 24.99,
      features: [
        'Everything in Basic',
        'Unlimited meetings',
        'Up to 50 attendees per meeting',
        'Recurring meetings',
        'Basic analytics',
        'Custom branding'
      ],
      limits: {
        maxMeetingsPerMonth: 1000,
        maxAttendeesPerMeeting: 50,
        roomBooking: true,
        recurringMeetings: true,
        customBranding: true,
        analytics: true,
        prioritySupport: false
      }
    },
    enterprise: {
      name: 'Enterprise',
      price: 99.99,
      features: [
        'Everything in Professional',
        'Unlimited meetings',
        'Unlimited attendees',
        'Advanced analytics',
        'Priority support',
        'Custom integrations',
        'Dedicated account manager'
      ],
      limits: {
        maxMeetingsPerMonth: Infinity,
        maxAttendeesPerMeeting: 500,
        roomBooking: true,
        recurringMeetings: true,
        customBranding: true,
        analytics: true,
        prioritySupport: true
      }
    }
  };

  /**
   * Create a new subscription
   * @param {string} organizationId - The organization ID
   * @param {string} plan - The subscription plan
   * @param {Object} paymentDetails - Payment method details
   * @param {string} billingCycle - 'monthly' or 'yearly'
   * @returns {Promise<Object>} - The created subscription
   */
  static async createSubscription(organizationId, plan, paymentDetails, billingCycle = 'monthly') {
    // Verify plan exists
    if (!this.PLANS[plan]) {
      throw new Error(`Invalid plan: ${plan}`);
    }

    // Calculate end date based on billing cycle
    const startDate = new Date();
    const endDate = new Date(startDate);
    if (billingCycle === 'monthly') {
      endDate.setMonth(endDate.getMonth() + 1);
    } else {
      endDate.setFullYear(endDate.getFullYear() + 1);
    }

    // Get price based on plan and billing cycle
    let price = this.PLANS[plan].price;
    if (billingCycle === 'yearly') {
      // Apply 20% discount for yearly billing
      price = price * 12 * 0.8;
    }

    // Create the subscription
    const subscription = await Subscription.create({
      organizationId,
      plan,
      status: 'active',
      startDate,
      endDate,
      billingCycle,
      price,
      paymentMethod: paymentDetails,
      billingHistory: [{
        date: startDate,
        amount: price,
        status: 'paid'
      }]
    });

    // Update organization with plan limits
    await Organization.findByIdAndUpdate(organizationId, {
      plan,
      usageLimits: {
        maxMeetingsPerMonth: this.PLANS[plan].limits.maxMeetingsPerMonth,
        maxAttendeesPerMeeting: this.PLANS[plan].limits.maxAttendeesPerMeeting
      }
    });

    return subscription;
  }

  /**
   * Check if an organization can schedule a meeting based on their limits
   * @param {string} organizationId - The organization ID
   * @param {number} attendeeCount - Number of attendees for the meeting
   * @returns {Promise<Object>} - Result with allowed status and reason if not allowed
   */
  static async canScheduleMeeting(organizationId, attendeeCount) {
    const organization = await Organization.findById(organizationId);
    if (!organization) {
      throw new Error(`Organization not found: ${organizationId}`);
    }

    if (!organization.active) {
      return {
        allowed: false,
        reason: 'Organization account is inactive'
      };
    }

    // Check attendee limit
    if (attendeeCount > organization.usageLimits.maxAttendeesPerMeeting) {
      return {
        allowed: false,
        reason: `Your plan allows a maximum of ${organization.usageLimits.maxAttendeesPerMeeting} attendees per meeting`
      };
    }

    // Check monthly meeting limit
    const currentMonth = new Date();
    currentMonth.setDate(1);
    currentMonth.setHours(0, 0, 0, 0);

    const meetingsThisMonth = await Meeting.countDocuments({
      organizationId,
      createdAt: { $gte: currentMonth }
    });

    if (meetingsThisMonth >= organization.usageLimits.maxMeetingsPerMonth) {
      return {
        allowed: false,
        reason: `You've reached your plan's limit of ${organization.usageLimits.maxMeetingsPerMonth} meetings per month`
      };
    }

    return {
      allowed: true
    };
  }

  /**
   * Record a meeting for usage tracking
   * @param {string} organizationId - The organization ID
   * @param {Object} meetingDetails - Details of the scheduled meeting
   * @returns {Promise<Object>} - The recorded meeting
   */
  static async recordMeeting(organizationId, meetingDetails) {
    // Create meeting record
    const meeting = await Meeting.create({
      organizationId,
      ...meetingDetails
    });

    // Update organization usage stats
    await Organization.findByIdAndUpdate(organizationId, {
      $inc: { 'usageStats.totalMeetingsScheduled': 1 },
      $set: { 'usageStats.lastMeetingScheduled': new Date() }
    });

    // Log this usage action
    await UsageLog.create({
      organizationId,
      action: 'meeting_scheduled',
      details: { meetingId: meeting._id }
    });

    return meeting;
  }
}

module.exports = SubscriptionService;
```

**services/billing.service.js**
```javascript
// services/billing.service.js
const stripe = require('stripe')(process.env.STRIPE_SECRET_KEY);
const { Organization, Subscription } = require('../models');
const SubscriptionService = require('./subscription.service');

class BillingService {
  /**
   * Create a Stripe customer for a new organization
   * @param {Object} organization - The organization object
   * @param {Object} user - The admin user
   * @returns {Promise<String>} - The Stripe customer ID
   */
  static async createCustomer(organization, user) {
    try {
      const customer = await stripe.customers.create({
        name: organization.name,
        email: user.email,
        metadata: {
          organizationId: organization._id.toString()
        }
      });

      // Update organization with Stripe customer ID
      await Organization.findByIdAndUpdate(
        organization._id,
        { stripeCustomerId: customer.id }
      );

      return customer.id;
    } catch (error) {
      console.error('Error creating Stripe customer:', error);
      throw new Error(`Failed to create customer: ${error.message}`);
    }
  }

  /**
   * Create a checkout session for subscription
   * @param {Object} organization - The organization
   * @param {String} plan - The subscription plan
   * @param {String} billingCycle - 'monthly' or 'yearly'
   * @param {String} successUrl - Redirect URL after successful payment
   * @param {String} cancelUrl - Redirect URL after canceled payment
   * @returns {Promise<Object>} - Checkout session
   */
  static async createCheckoutSession(organization, plan, billingCycle, successUrl, cancelUrl) {
    try {
      // Get price ID based on plan and billing cycle
      const priceId = await this.getPriceId(plan, billingCycle);

      if (!priceId) {
        throw new Error(`Invalid plan or billing cycle: ${plan} - ${billingCycle}`);
      }

      // Create checkout session
      const session = await stripe.checkout.sessions.create({
        customer: organization.stripeCustomerId,
        payment_method_types: ['card'],
        line_items: [
          {
            price: priceId,
            quantity: 1,
          },
        ],
        mode: 'subscription',
        subscription_data: {
          metadata: {
            organizationId: organization._id.toString(),
            plan: plan,
            billingCycle: billingCycle
          }
        },
        success_url: successUrl,
        cancel_url: cancelUrl,
        metadata: {
          organizationId: organization._id.toString()
        }
      });

      return session;
    } catch (error) {
      console.error('Error creating checkout session:', error);
      throw new Error(`Failed to create checkout session: ${error.message}`);
    }
  }

  /**
   * Handle webhook events from Stripe
   * @param {Object} event - Stripe webhook event
   * @returns {Promise<void>}
   */
  static async handleWebhookEvent(event) {
    try {
      switch (event.type) {
        case 'checkout.session.completed':
          await this.handleCheckoutSessionCompleted(event.data.object);
          break;
        
        case 'invoice.paid':
          await this.handleInvoicePaid(event.data.object);
          break;
        
        case 'invoice.payment_failed':
          await this.handleInvoicePaymentFailed(event.data.object);
          break;
        
        case 'customer.subscription.updated':
          await this.handleSubscriptionUpdated(event.data.object);
          break;
        
        case 'customer.subscription.deleted':
          await this.handleSubscriptionDeleted(event.data.object);
          break;
        
        default:
          console.log(`Unhandled event type: ${event.type}`);
      }
    } catch (error) {
      console.error(`Error handling webhook ${event.type}:`, error);
      throw error;
    }
  }
}

module.exports = BillingService;
```

### 4. Routes and Controllers

**routes/billing.routes.js**
```javascript
// routes/billing.routes.js
const express = require('express');
const router = express.Router();
const BillingController = require('../controllers/billing.controller');
const { isAuthenticated, isOrganizationAdmin } = require('../middleware/auth');

// Public routes
router.get('/plans', BillingController.showPlans);

// Authentication required routes
router.post('/select-plan', isAuthenticated, BillingController.selectPlan);
router.get('/success', isAuthenticated, BillingController.checkoutSuccess);
router.get('/subscription', isAuthenticated, BillingController.showSubscription);
router.get('/manage', isAuthenticated, isOrganizationAdmin, BillingController.manageSubscription);
router.get('/invoice/:invoiceId', isAuthenticated, BillingController.getInvoice);

// Webhook - no authentication required but verified with Stripe signature
router.post('/webhook', express.raw({ type: 'application/json' }), BillingController.handleWebhook);

module.exports = router;
```

**controllers/billing.controller.js**
```javascript
// controllers/billing.controller.js
const BillingService = require('../services/billing.service');
const SubscriptionService = require('../services/subscription.service');
const { Organization, Subscription } = require('../models');
const stripe = require('stripe')(process.env.STRIPE_SECRET_KEY);

/**
 * Billing controller for handling subscription and payment operations
 */
class BillingController {
  /**
   * Display pricing plans page
   * @param {Object} req - Express request object
   * @param {Object} res - Express response object
   */
  static async showPlans(req, res) {
    try {
      const plans = SubscriptionService.PLANS;
      res.render('billing/plans', {
        plans,
        currentPlan: req.user?.organization?.plan || 'free',
        stripePublishableKey: process.env.STRIPE_PUBLISHABLE_KEY
      });
    } catch (error) {
      console.error('Error showing plans:', error);
      res.status(500).send('Error loading pricing plans');
    }
  }

  /**
   * Handle plan selection and redirect to checkout
   * @param {Object} req - Express request object
   * @param {Object} res - Express response object
   */
  static async selectPlan(req, res) {
    try {
      const { plan, billingCycle } = req.body;
      const organizationId = req.user.organizationId;

      // Validate plan and billing cycle
      if (!['basic', 'pro', 'enterprise'].includes(plan)) {
        return res.status(400).send('Invalid plan selected');
      }

      if (!['monthly', 'yearly'].includes(billingCycle)) {
        return res.status(400).send('Invalid billing cycle');
      }

      // Get organization
      const organization = await Organization.findById(organizationId);
      if (!organization) {
        return res.status(404).send('Organization not found');
      }

      // Create Stripe customer if doesn't exist
      if (!organization.stripeCustomerId) {
        await BillingService.createCustomer(organization, req.user);
      }

      // Create checkout session
      const successUrl = `${req.protocol}://${req.get('host')}/billing/success?session_id={CHECKOUT_SESSION_ID}`;
      const cancelUrl = `${req.protocol}://${req.get('host')}/billing/plans`;

      const session = await BillingService.createCheckoutSession(
        organization,
        plan,
        billingCycle,
        successUrl,
        cancelUrl
      );

      // Redirect to checkout
      res.redirect(session.url);
    } catch (error) {
      console.error('Error selecting plan:', error);
      res.status(500).send(`Error selecting plan: ${error.message}`);
    }
  }

  /**
   * Handle Stripe webhooks
   * @param {Object} req - Express request object
   * @param {Object} res - Express response object
   */
  static async handleWebhook(req, res) {
    try {
      const signature = req.headers['stripe-signature'];
      let event;
      
      // Verify webhook signature
      try {
        event = stripe.webhooks.constructEvent(
          req.rawBody,
          signature,
          process.env.STRIPE_WEBHOOK_SECRET
        );
      } catch (err) {
        console.error('Webhook signature verification failed:', err.message);
        return res.status(400).send(`Webhook Error: ${err.message}`);
      }
      
      // Handle the event
      await BillingService.handleWebhookEvent(event);
      
      // Return success response
      res.json({ received: true });
    } catch (error) {
      console.error('Error handling webhook:', error);
      res.status(500).send(`Webhook Error: ${error.message}`);
    }
  }
}

module.exports = BillingController;
```

### 5. Slack Integration

**services/slack.service.js**
```javascript
// app.js - Main application file with multi-tenant support
require('dotenv').config();
const express = require('express');
const { App, ExpressReceiver } = require('@slack/bolt');
const { google } = require('googleapis');
const bodyParser = require('body-parser');
const session = require('express-session');
const mongoose = require('mongoose');
const passport = require('passport');
const LocalStrategy = require('passport-local').Strategy;
const bcrypt = require('bcryptjs');
const rateLimit = require('express-rate-limit');
const helmet = require('helmet');
```

## Environment Variables

```
# Server Configuration
PORT=3000
NODE_ENV=development

# MongoDB Connection
MONGODB_URI=mongodb://localhost:27017/calendarly
MONGODB_URI_PROD=mongodb+srv://username:password@cluster.mongodb.net/calendarly

# Session Configuration
SESSION_SECRET=your_session_secret_key_here

# JWT Configuration
JWT_SECRET=your_jwt_secret_here
JWT_EXPIRES_IN=7d

# Slack API Credentials
SLACK_CLIENT_ID=your_slack_client_id
SLACK_CLIENT_SECRET=your_slack_client_secret
SLACK_SIGNING_SECRET=your_slack_signing_secret
SLACK_APP_TOKEN=xapp-your-app-token-here
SLACK_BOT_TOKEN=xoxb-your-bot-token-here

# Google API Credentials
GOOGLE_CLIENT_ID=your_google_client_id
GOOGLE_CLIENT_SECRET=your_google_client_secret
GOOGLE_REDIRECT_URI=http://localhost:3000/auth/google/callback
GOOGLE_API_KEY=your_google_api_key

# Gemini API
GEMINI_API_KEY=your_gemini_api_key

# Stripe API Keys
STRIPE_PUBLISHABLE_KEY=pk_test_your_publishable_key
STRIPE_SECRET_KEY=sk_test_your_secret_key
STRIPE_WEBHOOK_SECRET=whsec_your_webhook_secret

# Email Configuration (for notifications)
EMAIL_SERVICE=gmail
EMAIL_USER=your_email@gmail.com
EMAIL_PASSWORD=your_email_app_password
EMAIL_FROM=Calendarly <noreply@calendarly.com>

# Admin User Setup
ADMIN_EMAIL=admin@yourdomain.com
ADMIN_PASSWORD=secure_initial_password

# Default Meeting Rooms (JSON array)
MEETING_ROOMS=[{"email":"room1@example.com","name":"Conference Room A","capacity":10,"location":"Floor 1"},{"email":"room2@example.com","name":"Conference Room B","capacity":6,"location":"Floor 1"}]

# Feature Flags
ENABLE_ROOM_BOOKING=true
ENABLE_SLACK_COMMANDS=true
ENABLE_ANALYTICS=true

# API Rate Limiting
RATE_LIMIT_WINDOW_MS=900000
RATE_LIMIT_MAX_REQUESTS=100

# Logging
LOG_LEVEL=info
```

## Pricing Strategy (Indian Rupees)

| Feature | CALENDAR LITE | CALENDAR PRO | CALENDAR TEAMS | CALENDAR ENTERPRISE |
|---------|--------------|-------------|----------------|---------------------|
| **Monthly Price** | ₹499 | ₹1,249 | ₹2,999 | ₹7,499 |
| **Annual Price** (per month) | ₹399 (₹4,788/yr) | ₹999 (₹11,988/yr) | ₹2,249 (₹26,988/yr) | ₹5,624 (₹67,488/yr) |
| **AI-Scheduled Meetings** | 20/month | 100/month | 500/month | 2,500/month |
| **Participants per Meeting** | 5 max | 15 max | 30 max | 100 max |
| **Meeting History** | 30 days | 90 days | 1 year | Unlimited |

### Cost & Profit Analysis

| Cost & Profit Analysis | CALENDAR LITE | CALENDAR PRO | CALENDAR TEAMS | CALENDAR ENTERPRISE |
|----------------------|--------------|-------------|----------------|---------------------|
| **Monthly Price (INR)** | ₹499 | ₹1,249 | ₹2,999 | ₹7,499 |
| **Total Cost (Monthly)** | ₹47 | ₹122 | ₹380 | ₹1,460 |
| **Profit per user (Monthly)** | ₹452 | ₹1,127 | ₹2,619 | ₹6,039 |
| **Profit Margin (Monthly)** | 90.6% | 90.2% | 87.3% | 80.5% |

## Integrations Overview

### Gemini AI Integration

The application uses Google's Gemini API to process natural language scheduling requests:

```javascript
// Example: Integrating with Gemini API for scheduling intent analysis
const { GoogleGenerativeAI } = require('@google/generative-ai');

// Initialize the Gemini API client
const genAI = new GoogleGenerativeAI(process.env.GEMINI_API_KEY);
const model = genAI.getGenerativeModel({ model: 'gemini-pro' });

async function analyzeSchedulingIntent(messageText) {
  try {
    const prompt = `
      Extract scheduling information from this message:
      "${messageText}"
      
      Provide the following details in JSON format:
      - meeting_purpose: The purpose or title of the meeting
      - participants: Array of mentioned participants
      - date: Preferred date(s)
      - time: Preferred time(s)
      - duration: Meeting duration in minutes
      - location_preference: Any location or room preferences
    `;

    const result = await model.generateContent(prompt);
    const response = await result.response;
    const textResponse = response.text();
    
    // Parse JSON from the response
    const jsonMatch = textResponse.match(/```json\n([\s\S]*?)\n```/);
    if (jsonMatch && jsonMatch[1]) {
      return JSON.parse(jsonMatch[1]);
    }
    
    return null;
  } catch (error) {
    console.error('Gemini API error:', error);
    throw new Error(`Failed to analyze scheduling intent: ${error.message}`);
  }
}
```

### Google Calendar Integration

The application interacts with Google Calendar through Google's API client:

```javascript
// Example: Integrating with Google Calendar MCP Server
const { google } = require('googleapis');

// Initialize OAuth2 client
const oAuth2Client = new google.auth.OAuth2(
  process.env.GOOGLE_CLIENT_ID,
  process.env.GOOGLE_CLIENT_SECRET,
  process.env.GOOGLE_REDIRECT_URI
);

async function findAvailableTime(participants, dateRange, duration) {
  try {
    // Set credentials for the organization
    oAuth2Client.setCredentials({
      refresh_token: process.env.GOOGLE_REFRESH_TOKEN
    });

    // Create calendar client
    const calendar = google.calendar({ version: 'v3', auth: oAuth2Client });
    
    // Format participants for freebusy query
    const items = participants.map(email => ({ id: email }));
    
    // Calculate start and end times
    const startTime = new Date(dateRange.start);
    const endTime = new Date(dateRange.end);
    
    // Query freeBusy via MCP server
    const freeBusyQuery = await calendar.freebusy.query({
      requestBody: {
        timeMin: startTime.toISOString(),
        timeMax: endTime.toISOString(),
        items: items,
        // MCP specific parameters
        calendarExpansionMax: 50,
        groupExpansionMax: 100
      }
    });
    
    // Process free/busy information
    const busyTimes = freeBusyQuery.data.calendars;
    
    // Find available slots using time optimization algorithm
    const availableSlots = findAvailableSlots(busyTimes, startTime, endTime, duration);
    
    return availableSlots;
  } catch (error) {
    console.error('MCP Calendar error:', error);
    throw new Error(`Failed to find available time: ${error.message}`);
  }
}
```

## Deployment Instructions

1. **Clone the repository**
   ```bash
   git clone https://github.com/yourusername/calendarly.git
   cd calendarly
   ```

2. **Install dependencies**
   ```bash
   npm install
   ```

3. **Configure environment variables**
   ```bash
   cp .env.example .env
   # Edit .env file with your credentials
   ```

4. **Set up the database**
   ```bash
   # Ensure MongoDB is running
   # Create database and initial admin user
   npm run setup-db
   ```

5. **Start the application**
   ```bash
   # Development mode
   npm run dev
   
   # Production mode
   npm start
   ```

## Business Model Summary

Calendarly offers a tiered subscription model with the following options:

1. **Free Tier** - Limited to 3 meetings/month (acquisition channel)
2. **Calendar Lite** (₹499/month) - 20 meetings/month, 5 attendees per meeting
3. **Calendar Pro** (₹1,249/month) - 100 meetings/month, 15 attendees per meeting
4. **Calendar Teams** (₹2,999/month) - 500 meetings/month, 30 attendees per meeting
5. **Calendar Enterprise** (₹7,499/month) - 2,500 meetings/month, 100 attendees per meeting

The business model achieves profitability through:
- High profit margins (80-90% across all tiers)
- Annual billing discounts to improve cash flow
- Add-on services for additional revenue
- Team plans for organizational adoption

Cost optimization strategies include:
- Efficient Gemini API token usage
- Request caching
- Multi-tenant architecture
- Graduated infrastructure scaling

Break-even occurs at approximately 40 paying users with the expected tier distribution.
