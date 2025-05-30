# Testing Strategy & Quality Assurance Plan
## Torvan Medical CleanStation Production Workflow Digitalization

**Version:** 1.0  
**Date:** December 2024  
**Document Type:** Testing Strategy Specification  
**Scope:** Complete System Testing Framework

---

## Executive Summary

This document establishes a comprehensive testing strategy for the CleanStation digitalization project, ensuring high-quality delivery through systematic validation of all system components. The strategy encompasses unit testing, integration testing, end-to-end testing, performance validation, security assessment, and quality assurance processes.

### Testing Objectives
- **Quality Assurance** - Ensure system meets all functional requirements
- **Reliability Validation** - Verify system stability under production conditions
- **Performance Verification** - Confirm system meets performance requirements
- **Security Assessment** - Validate security controls and compliance
- **User Experience Validation** - Ensure usability across all user roles
- **Compliance Verification** - Confirm regulatory requirement adherence

### Testing Coverage Goals
- **Unit Tests:** 90%+ code coverage
- **Integration Tests:** 100% API endpoint coverage
- **E2E Tests:** 100% critical user journey coverage
- **Performance Tests:** All performance requirements validated
- **Security Tests:** All OWASP top 10 vulnerabilities covered
- **Accessibility Tests:** WCAG 2.1 AA compliance verified

---

## Testing Framework Architecture

### Technology Stack
```typescript
interface TestingStack {
  unitTesting: {
    framework: 'Jest';
    utilities: ['@testing-library/react', '@testing-library/jest-dom'];
    coverage: 'jest-coverage';
    mocking: 'jest.mock()';
  };
  
  integrationTesting: {
    framework: 'Jest + Supertest';
    database: 'Test containers';
    apiTesting: 'Supertest';
    mocking: 'MSW (Mock Service Worker)';
  };
  
  e2eTesting: {
    framework: 'Playwright';
    browsers: ['Chromium', 'Firefox', 'Safari'];
    devices: 'Mobile + Desktop';
    visual: 'Playwright screenshots';
  };
  
  performanceTesting: {
    load: 'Artillery.js';
    stress: 'K6';
    monitoring: 'Lighthouse CI';
    profiling: 'Node.js profiler';
  };
  
  securityTesting: {
    static: 'SonarQube';
    dynamic: 'OWASP ZAP';
    dependencies: 'npm audit + Snyk';
    penetration: 'Custom security suite';
  };
  
  accessibility: {
    automated: 'axe-core';
    manual: 'Screen reader testing';
    compliance: 'WAVE + Pa11y';
  };
}
```

### Test Environment Strategy
```typescript
interface TestEnvironments {
  unit: {
    description: 'Isolated component testing';
    database: 'In-memory SQLite';
    services: 'Mocked';
    data: 'Factory-generated';
  };
  
  integration: {
    description: 'API and service integration';
    database: 'PostgreSQL test container';
    services: 'Real implementations';
    data: 'Seeded test data';
  };
  
  staging: {
    description: 'Production-like environment';
    database: 'PostgreSQL (isolated)';
    services: 'Full stack deployed';
    data: 'Anonymized production data';
  };
  
  performance: {
    description: 'Load and stress testing';
    database: 'Production-spec PostgreSQL';
    services: 'Scaled infrastructure';
    data: 'Large dataset simulation';
  };
}
```

---

## Unit Testing Strategy

### Component Testing Standards
```typescript
// React component testing example
describe('OrderCreationWizard', () => {
  beforeEach(() => {
    jest.clearAllMocks();
  });

  test('renders all wizard steps correctly', () => {
    render(<OrderCreationWizard />);
    
    expect(screen.getByText('Step 1: Customer Information')).toBeInTheDocument();
    expect(screen.getByText('Step 2: Sink Configuration')).toBeInTheDocument();
    expect(screen.getByText('Step 3: Faucet Selection')).toBeInTheDocument();
    expect(screen.getByText('Step 4: Accessories')).toBeInTheDocument();
    expect(screen.getByText('Step 5: Review & Submit')).toBeInTheDocument();
  });

  test('validates customer information correctly', async () => {
    const onSubmit = jest.fn();
    render(<OrderCreationWizard onSubmit={onSubmit} />);
    
    // Test validation
    fireEvent.click(screen.getByText('Next'));
    expect(screen.getByText('Customer name is required')).toBeInTheDocument();
    
    // Fill valid data
    fireEvent.change(screen.getByLabelText('Customer Name'), {
      target: { value: 'Test Customer' }
    });
    fireEvent.change(screen.getByLabelText('PO Number'), {
      target: { value: 'PO-12345' }
    });
    
    fireEvent.click(screen.getByText('Next'));
    expect(screen.getByText('Step 2: Sink Configuration')).toBeVisible();
  });

  test('generates BOM correctly based on selections', async () => {
    const mockBOMGeneration = jest.fn().mockResolvedValue(mockBOM);
    jest.mock('../services/bomService', () => ({
      generateBOM: mockBOMGeneration
    }));

    render(<OrderCreationWizard />);
    
    // Navigate through wizard with selections
    await completeWizardFlow();
    
    expect(mockBOMGeneration).toHaveBeenCalledWith({
      sinkType: 'MDRD-36',
      faucets: [{ type: 'sensor', count: 2 }],
      accessories: [{ type: 'soap-dispenser', count: 1 }]
    });
  });
});
```

### Service Layer Testing
```typescript
// Business logic testing
describe('BOMGenerationService', () => {
  let bomService: BOMGenerationService;
  let mockPartRepository: jest.Mocked<PartRepository>;

  beforeEach(() => {
    mockPartRepository = createMockPartRepository();
    bomService = new BOMGenerationService(mockPartRepository);
  });

  test('generates complete BOM for MDRD configuration', async () => {
    const config = createTestSinkConfiguration({
      sinkType: 'MDRD-36',
      faucets: [{ type: 'sensor', position: 'left' }],
      accessories: ['soap-dispenser']
    });

    const bom = await bomService.generateBOM(config);

    expect(bom.items).toHaveLength(expectedItemCount);
    expect(bom.totalCost).toBeGreaterThan(0);
    expect(bom.items.find(item => item.category === 'sink')).toBeDefined();
    expect(bom.items.find(item => item.category === 'faucet')).toBeDefined();
  });

  test('handles part substitutions correctly', async () => {
    mockPartRepository.findByPartNumber.mockResolvedValueOnce(null);
    mockPartRepository.findSubstitute.mockResolvedValueOnce(mockSubstitutePart);

    const config = createTestSinkConfiguration();
    const bom = await bomService.generateBOM(config);

    expect(bom.items.some(item => item.isSubstitute)).toBe(true);
    expect(bom.warnings).toContain('Part substitution applied');
  });
});
```

### Database Testing
```typescript
// Repository testing with test containers
describe('OrderRepository', () => {
  let repository: OrderRepository;
  let db: Database;

  beforeAll(async () => {
    db = await setupTestDatabase();
    repository = new OrderRepository(db);
  });

  afterAll(async () => {
    await teardownTestDatabase(db);
  });

  beforeEach(async () => {
    await seedTestData(db);
  });

  afterEach(async () => {
    await cleanupTestData(db);
  });

  test('creates order with correct relationships', async () => {
    const orderData = createTestOrderData();
    const order = await repository.create(orderData);

    expect(order.orderId).toBeDefined();
    expect(order.sinkConfiguration).toEqual(orderData.sinkConfiguration);
    
    // Verify database state
    const savedOrder = await repository.findById(order.orderId);
    expect(savedOrder).toEqual(order);
  });

  test('updates order status with audit trail', async () => {
    const order = await repository.create(createTestOrderData());
    
    await repository.updateStatus(order.orderId, 'production', 'user123');
    
    const updatedOrder = await repository.findById(order.orderId);
    expect(updatedOrder.status).toBe('production');
    
    const auditLog = await repository.getAuditTrail(order.orderId);
    expect(auditLog).toHaveLength(2); // Create + Update
  });
});
```

---

## Integration Testing Strategy

### API Integration Testing
```typescript
// API endpoint testing
describe('Order API Integration', () => {
  let app: Application;
  let authToken: string;

  beforeAll(async () => {
    app = await createTestApp();
    authToken = await getTestAuthToken('production-coordinator');
  });

  afterAll(async () => {
    await cleanupTestApp(app);
  });

  test('POST /api/orders creates order successfully', async () => {
    const orderData = createValidOrderData();

    const response = await request(app)
      .post('/api/orders')
      .set('Authorization', `Bearer ${authToken}`)
      .send(orderData)
      .expect(201);

    expect(response.body.data.orderId).toBeDefined();
    expect(response.body.data.status).toBe('created');
    expect(response.body.data.customerInfo).toEqual(orderData.customerInfo);
  });

  test('PUT /api/orders/:id requires proper authorization', async () => {
    const order = await createTestOrder();
    const unauthorizedToken = await getTestAuthToken('assembler');

    await request(app)
      .put(`/api/orders/${order.orderId}`)
      .set('Authorization', `Bearer ${unauthorizedToken}`)
      .send({ status: 'cancelled' })
      .expect(403);
  });

  test('GET /api/orders supports filtering and pagination', async () => {
    await createMultipleTestOrders(15);

    const response = await request(app)
      .get('/api/orders?status=created&limit=10&offset=0')
      .set('Authorization', `Bearer ${authToken}`)
      .expect(200);

    expect(response.body.data.orders).toHaveLength(10);
    expect(response.body.meta.total).toBeGreaterThan(10);
    expect(response.body.meta.hasMore).toBe(true);
  });
});
```

### Service Integration Testing
```typescript
// Cross-service integration testing
describe('QC Workflow Integration', () => {
  test('complete Pre-QC to Production workflow', async () => {
    // Create order
    const order = await orderService.create(createTestOrderData());
    
    // Generate and approve BOM
    const bom = await bomService.generateBOM(order.sinkConfiguration);
    await bomService.approve(bom.bomId, 'procurement-user');
    
    // Complete Pre-QC
    const preQCForm = await qcService.startPreQC(order.orderId, 'qc-user');
    const preQCResults = createPassingQCResults();
    await qcService.submitPreQC(preQCForm.formId, preQCResults);
    
    // Verify order status progression
    const updatedOrder = await orderService.getById(order.orderId);
    expect(updatedOrder.status).toBe('production');
    
    // Verify notifications sent
    const notifications = await notificationService.getForOrder(order.orderId);
    expect(notifications.some(n => n.type === 'preqc_completed')).toBe(true);
  });
});
```

### Database Integration Testing
```typescript
// Database transaction testing
describe('Database Transaction Integration', () => {
  test('BOM generation maintains data consistency', async () => {
    const order = await createTestOrder();
    
    // Simulate concurrent BOM generation attempts
    const promises = Array(5).fill(null).map(() => 
      bomService.generateBOM(order.sinkConfiguration)
    );
    
    const results = await Promise.allSettled(promises);
    const successful = results.filter(r => r.status === 'fulfilled');
    
    // Only one should succeed due to database constraints
    expect(successful).toHaveLength(1);
    
    // Verify no partial data corruption
    const boms = await bomRepository.findByOrderId(order.orderId);
    expect(boms).toHaveLength(1);
    expect(boms[0].items.length).toBeGreaterThan(0);
  });
});
```

---

## End-to-End Testing Strategy

### User Journey Testing
```typescript
// Complete workflow E2E testing
describe('Complete Order Lifecycle E2E', () => {
  test('Production Coordinator completes order workflow', async ({ page }) => {
    // Login as Production Coordinator
    await page.goto('/login');
    await page.fill('[data-testid=email]', 'coordinator@test.com');
    await page.fill('[data-testid=password]', 'testpassword');
    await page.click('[data-testid=login-button]');
    
    // Create new order
    await page.click('[data-testid=create-order-button]');
    
    // Step 1: Customer Information
    await page.fill('[data-testid=customer-name]', 'Test Hospital');
    await page.fill('[data-testid=po-number]', 'PO-12345');
    await page.click('[data-testid=next-button]');
    
    // Step 2: Sink Configuration
    await page.selectOption('[data-testid=sink-type]', 'MDRD-36');
    await page.check('[data-testid=single-user-sink]');
    await page.click('[data-testid=next-button]');
    
    // Step 3: Faucet Selection
    await page.click('[data-testid=add-faucet]');
    await page.selectOption('[data-testid=faucet-type-0]', 'sensor');
    await page.click('[data-testid=next-button]');
    
    // Step 4: Accessories
    await page.check('[data-testid=soap-dispenser]');
    await page.click('[data-testid=next-button]');
    
    // Step 5: Review & Submit
    await expect(page.locator('[data-testid=order-summary]')).toBeVisible();
    await page.click('[data-testid=submit-order]');
    
    // Verify order creation
    await expect(page.locator('[data-testid=success-message]')).toBeVisible();
    const orderId = await page.textContent('[data-testid=order-id]');
    expect(orderId).toBeTruthy();
  });

  test('QC Person completes Pre-QC workflow', async ({ page }) => {
    // Setup: Create order and BOM
    const order = await createTestOrderViaAPI();
    await approveBOMViaAPI(order.orderId);
    
    // Login as QC Person
    await loginAs(page, 'qc-person');
    
    // Navigate to Pre-QC queue
    await page.click('[data-testid=qc-menu]');
    await page.click('[data-testid=preqc-queue]');
    
    // Find and start Pre-QC for test order
    await page.click(`[data-testid=start-preqc-${order.orderId}]`);
    
    // Complete Pre-QC form
    await fillPreQCForm(page);
    await page.click('[data-testid=submit-preqc]');
    
    // Verify completion
    await expect(page.locator('[data-testid=preqc-success]')).toBeVisible();
    
    // Verify order moved to production
    const orderStatus = await getOrderStatusViaAPI(order.orderId);
    expect(orderStatus).toBe('production');
  });
});
```

### Cross-Browser Testing
```typescript
// Multi-browser compatibility testing
['chromium', 'firefox', 'webkit'].forEach(browserName => {
  test.describe(`${browserName} compatibility`, () => {
    test('order creation works across browsers', async ({ browser }) => {
      const context = await browser.newContext();
      const page = await context.newPage();
      
      await runOrderCreationWorkflow(page);
      
      await context.close();
    });
  });
});
```

### Mobile Responsiveness Testing
```typescript
// Mobile device testing
test.describe('Mobile responsiveness', () => {
  ['iPhone 12', 'iPad', 'Pixel 5'].forEach(device => {
    test(`works on ${device}`, async ({ browser }) => {
      const context = await browser.newContext({
        ...devices[device]
      });
      const page = await context.newPage();
      
      await testMobileWorkflow(page);
      
      await context.close();
    });
  });
});
```

---

## Performance Testing Strategy

### Load Testing
```typescript
// Artillery.js load testing configuration
const loadTestConfig = {
  config: {
    target: 'https://api.cleanstation.staging.com',
    phases: [
      { duration: 60, arrivalRate: 10 },  // Warm up
      { duration: 300, arrivalRate: 50 }, // Sustained load
      { duration: 120, arrivalRate: 100 } // Peak load
    ],
    payload: {
      path: './test-data/orders.csv',
      fields: ['customerId', 'poNumber', 'sinkType']
    }
  },
  scenarios: [
    {
      name: 'Order Creation Workflow',
      weight: 40,
      flow: [
        { post: { url: '/api/auth/login', json: { email: '{{ email }}', password: '{{ password }}' } } },
        { post: { url: '/api/orders', json: '{{ orderData }}' } },
        { get: { url: '/api/orders/{{ orderId }}' } }
      ]
    },
    {
      name: 'BOM Generation',
      weight: 30,
      flow: [
        { post: { url: '/api/orders/{{ orderId }}/bom/generate' } },
        { get: { url: '/api/bom/{{ bomId }}' } }
      ]
    },
    {
      name: 'QC Workflow',
      weight: 30,
      flow: [
        { post: { url: '/api/qc/forms', json: '{{ qcFormData }}' } },
        { put: { url: '/api/qc/forms/{{ formId }}/submit', json: '{{ qcResults }}' } }
      ]
    }
  ]
};
```

### Stress Testing
```typescript
// K6 stress testing
import http from 'k6/http';
import { check, sleep } from 'k6';

export let options = {
  stages: [
    { duration: '5m', target: 100 },  // Scale up
    { duration: '10m', target: 100 }, // Sustained stress
    { duration: '5m', target: 200 },  // Scale up further
    { duration: '10m', target: 200 }, // Peak stress
    { duration: '5m', target: 0 }     // Scale down
  ],
  thresholds: {
    http_req_duration: ['p(95)<2000'], // 95% of requests under 2s
    http_req_failed: ['rate<0.1'],     // Error rate under 10%
    http_reqs: ['rate>10']             // Request rate above 10/s
  }
};

export default function () {
  // Test order creation under stress
  const orderData = generateRandomOrderData();
  
  const response = http.post('https://api.cleanstation.com/api/orders', JSON.stringify(orderData), {
    headers: { 'Content-Type': 'application/json', 'Authorization': `Bearer ${__ENV.API_TOKEN}` }
  });
  
  check(response, {
    'status is 201': (r) => r.status === 201,
    'response time < 2000ms': (r) => r.timings.duration < 2000,
    'order ID present': (r) => JSON.parse(r.body).data.orderId !== undefined
  });
  
  sleep(1);
}
```

### Database Performance Testing
```typescript
// Database performance benchmarks
describe('Database Performance', () => {
  test('order queries perform within SLA', async () => {
    // Create large dataset
    await seedLargeDataset(10000); // 10k orders
    
    const startTime = performance.now();
    
    // Test complex query performance
    const orders = await orderRepository.findWithFilters({
      status: 'production',
      createdAfter: new Date('2024-01-01'),
      customerName: 'Test%',
      limit: 50
    });
    
    const endTime = performance.now();
    const queryTime = endTime - startTime;
    
    expect(queryTime).toBeLessThan(500); // Query under 500ms
    expect(orders.length).toBeGreaterThan(0);
  });

  test('concurrent writes maintain consistency', async () => {
    const promises = Array(50).fill(null).map((_, i) => 
      orderRepository.create(createTestOrderData(`Order-${i}`))
    );
    
    const results = await Promise.allSettled(promises);
    const successful = results.filter(r => r.status === 'fulfilled').length;
    
    expect(successful).toBe(50); // All writes successful
    
    // Verify data integrity
    const orderCount = await orderRepository.count();
    expect(orderCount).toBeGreaterThanOrEqual(50);
  });
});
```

---

## Security Testing Strategy

### Authentication & Authorization Testing
```typescript
// Security testing for auth endpoints
describe('Authentication Security', () => {
  test('prevents brute force attacks', async () => {
    const attempts = Array(10).fill(null).map(() =>
      request(app)
        .post('/api/auth/login')
        .send({ email: 'test@test.com', password: 'wrongpassword' })
    );
    
    const results = await Promise.all(attempts);
    
    // Should rate limit after 5 attempts
    expect(results.slice(5).every(r => r.status === 429)).toBe(true);
  });

  test('JWT tokens have proper expiration', async () => {
    const { accessToken } = await login('test@test.com', 'password');
    const decoded = jwt.decode(accessToken) as any;
    
    expect(decoded.exp).toBeDefined();
    expect(decoded.exp - decoded.iat).toBeLessThanOrEqual(3600); // 1 hour max
  });

  test('role-based access control works correctly', async () => {
    const assemblerToken = await getTokenForRole('assembler');
    
    // Assembler should not access admin endpoints
    await request(app)
      .get('/api/admin/users')
      .set('Authorization', `Bearer ${assemblerToken}`)
      .expect(403);
  });
});
```

### Input Validation Security
```typescript
// SQL injection and XSS prevention
describe('Input Validation Security', () => {
  test('prevents SQL injection in search queries', async () => {
    const maliciousInput = "'; DROP TABLE orders; --";
    
    const response = await request(app)
      .get(`/api/orders?search=${encodeURIComponent(maliciousInput)}`)
      .set('Authorization', `Bearer ${validToken}`)
      .expect(400); // Should reject malicious input
    
    // Verify database integrity
    const orderCount = await orderRepository.count();
    expect(orderCount).toBeGreaterThan(0); // Table still exists
  });

  test('sanitizes HTML input to prevent XSS', async () => {
    const xssPayload = '<script>alert("xss")</script>';
    
    const response = await request(app)
      .post('/api/orders')
      .set('Authorization', `Bearer ${validToken}`)
      .send({
        customerInfo: { name: xssPayload },
        // ... other valid data
      })
      .expect(400); // Should reject XSS payload
  });
});
```

### Dependency Security Testing
```typescript
// Automated dependency vulnerability scanning
describe('Dependency Security', () => {
  test('no high-severity vulnerabilities in dependencies', async () => {
    const auditResult = await runNpmAudit();
    
    expect(auditResult.vulnerabilities.high).toBe(0);
    expect(auditResult.vulnerabilities.critical).toBe(0);
  });

  test('all dependencies are up to date', async () => {
    const outdatedPackages = await checkOutdatedPackages();
    
    // Allow minor version differences, but no major version gaps
    const majorOutdated = outdatedPackages.filter(pkg => 
      semver.major(pkg.current) < semver.major(pkg.latest)
    );
    
    expect(majorOutdated).toHaveLength(0);
  });
});
```

---

## Accessibility Testing Strategy

### Automated Accessibility Testing
```typescript
// axe-core integration testing
describe('Accessibility Compliance', () => {
  test('order creation wizard is fully accessible', async ({ page }) => {
    await page.goto('/orders/create');
    
    const accessibilityResults = await new AxeBuilder({ page })
      .withTags(['wcag2a', 'wcag2aa'])
      .analyze();
    
    expect(accessibilityResults.violations).toHaveLength(0);
  });

  test('keyboard navigation works throughout application', async ({ page }) => {
    await page.goto('/dashboard');
    
    // Test keyboard navigation
    await page.keyboard.press('Tab');
    let focusedElement = await page.evaluate(() => document.activeElement?.tagName);
    expect(['BUTTON', 'A', 'INPUT']).toContain(focusedElement);
    
    // Continue tabbing and verify logical order
    for (let i = 0; i < 10; i++) {
      await page.keyboard.press('Tab');
      const currentElement = await page.evaluate(() => ({
        tag: document.activeElement?.tagName,
        text: document.activeElement?.textContent?.trim(),
        id: document.activeElement?.id
      }));
      
      expect(currentElement.tag).toBeTruthy();
    }
  });
});
```

### Screen Reader Testing
```typescript
// Screen reader compatibility testing
describe('Screen Reader Compatibility', () => {
  test('form labels are properly associated', async ({ page }) => {
    await page.goto('/orders/create');
    
    // Check aria-label and label associations
    const inputs = await page.locator('input').all();
    
    for (const input of inputs) {
      const hasLabel = await input.evaluate(el => {
        const id = el.id;
        const ariaLabel = el.getAttribute('aria-label');
        const ariaLabelledBy = el.getAttribute('aria-labelledby');
        const label = document.querySelector(`label[for="${id}"]`);
        
        return !!(ariaLabel || ariaLabelledBy || label);
      });
      
      expect(hasLabel).toBe(true);
    }
  });

  test('dynamic content announces properly', async ({ page }) => {
    await page.goto('/orders/create');
    
    // Add aria-live region monitoring
    await page.addScriptTag({
      content: `
        const liveRegions = document.querySelectorAll('[aria-live]');
        window.announcements = [];
        liveRegions.forEach(region => {
          const observer = new MutationObserver(mutations => {
            mutations.forEach(mutation => {
              if (mutation.type === 'childList' || mutation.type === 'characterData') {
                window.announcements.push(region.textContent);
              }
            });
          });
          observer.observe(region, { childList: true, characterData: true, subtree: true });
        });
      `
    });
    
    // Trigger form validation
    await page.click('[data-testid=next-button]');
    
    // Check announcements
    const announcements = await page.evaluate(() => window.announcements);
    expect(announcements.some(a => a.includes('required'))).toBe(true);
  });
});
```

---

## Test Data Management

### Test Data Factory
```typescript
// Centralized test data generation
class TestDataFactory {
  static createOrder(overrides: Partial<Order> = {}): Order {
    return {
      orderId: faker.datatype.uuid(),
      poNumber: faker.random.alphaNumeric(8),
      customerInfo: this.createCustomerInfo(),
      sinkConfiguration: this.createSinkConfiguration(),
      buildNumbers: [faker.random.alphaNumeric(6)],
      status: 'created',
      createdAt: faker.date.recent(),
      updatedAt: faker.date.recent(),
      ...overrides
    };
  }

  static createSinkConfiguration(): SinkConfiguration {
    return {
      sinkType: faker.helpers.arrayElement(['MDRD-36', 'MDRD-48', 'MDRD-60']),
      dimensions: {
        width: faker.datatype.number({ min: 30, max: 72 }),
        depth: faker.datatype.number({ min: 18, max: 24 }),
        height: faker.datatype.number({ min: 34, max: 38 })
      },
      faucetConfiguration: [this.createFaucetConfig()],
      sprayerConfiguration: [],
      accessories: [this.createAccessoryConfig()],
      customizations: []
    };
  }

  static createQCResults(passing: boolean = true): QCResult[] {
    return [
      {
        checkId: faker.datatype.uuid(),
        description: 'Visual inspection of sink finish',
        expectedValue: 'Smooth, uniform finish',
        actualValue: passing ? 'Smooth, uniform finish' : 'Minor scratches observed',
        result: passing ? 'Pass' : 'Fail',
        notes: passing ? '' : 'Requires rework'
      }
    ];
  }
}
```

### Database Seeding
```typescript
// Test database seeding utilities
class TestDatabaseSeeder {
  async seedBasicData() {
    // Seed users
    await this.seedUsers();
    
    // Seed catalog data
    await this.seedParts();
    await this.seedAssemblies();
    await this.seedCategories();
    
    // Seed QC templates
    await this.seedQCTemplates();
  }

  async seedLargeDataset(orderCount: number) {
    const orders = Array(orderCount).fill(null).map(() => 
      TestDataFactory.createOrder()
    );
    
    await this.db.batchInsert('orders', orders);
    
    // Seed related data
    for (const order of orders) {
      if (Math.random() > 0.5) {
        await this.seedBOMForOrder(order.orderId);
      }
      if (Math.random() > 0.7) {
        await this.seedQCResultsForOrder(order.orderId);
      }
    }
  }

  async cleanupTestData() {
    const tables = ['qc_results', 'bom_items', 'boms', 'orders'];
    
    for (const table of tables) {
      await this.db.raw(`DELETE FROM ${table} WHERE created_at > NOW() - INTERVAL '1 hour'`);
    }
  }
}
```

---

## Test Execution & Reporting

### Continuous Integration Pipeline
```yaml
# GitHub Actions testing workflow
name: Test Suite
on: [push, pull_request]

jobs:
  unit-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      - run: npm ci
      - run: npm run test:unit
      - uses: codecov/codecov-action@v3

  integration-tests:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_PASSWORD: test
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
      - run: npm ci
      - run: npm run test:integration

  e2e-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
      - run: npm ci
      - run: npx playwright install
      - run: npm run test:e2e

  performance-tests:
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v3
      - run: npm ci
      - run: npm run test:performance
      - uses: actions/upload-artifact@v3
        with:
          name: performance-results
          path: performance-results/
```

### Test Reporting
```typescript
// Custom test reporter
class CleanStationTestReporter {
  generateReport(testResults: TestResults) {
    return {
      summary: {
        total: testResults.numTotalTests,
        passed: testResults.numPassedTests,
        failed: testResults.numFailedTests,
        coverage: testResults.coverageMap.getCoverageSummary()
      },
      
      failureAnalysis: this.analyzeFailures(testResults.testResults),
      performanceMetrics: this.extractPerformanceMetrics(testResults),
      accessibilityReport: this.generateAccessibilityReport(testResults),
      
      recommendations: this.generateRecommendations(testResults)
    };
  }

  generateAccessibilityReport(testResults: TestResults) {
    const a11yResults = testResults.testResults
      .filter(test => test.testFilePath.includes('accessibility'))
      .map(test => ({
        component: this.extractComponentName(test.testFilePath),
        violations: this.extractA11yViolations(test),
        wcagLevel: this.determineWCAGCompliance(test)
      }));

    return {
      overallCompliance: this.calculateOverallCompliance(a11yResults),
      componentResults: a11yResults,
      criticalIssues: a11yResults.filter(r => r.violations.length > 0)
    };
  }
}
```

---

## Quality Gates & Criteria

### Definition of Done
```typescript
interface QualityGates {
  unitTests: {
    coverage: '>= 90%';
    passing: '100%';
    performant: 'All tests < 5s execution time';
  };
  
  integrationTests: {
    apiCoverage: '100% of endpoints tested';
    dbIntegrity: 'All transactions tested';
    crossService: 'All service integrations verified';
  };
  
  e2eTests: {
    userJourneys: '100% critical paths tested';
    crossBrowser: 'Chrome, Firefox, Safari verified';
    responsive: 'Mobile and desktop tested';
  };
  
  performance: {
    loadTime: 'Page load < 3s';
    apiResponse: 'API response < 500ms p95';
    concurrent: '50 concurrent users supported';
  };
  
  accessibility: {
    wcagCompliance: 'WCAG 2.1 AA compliance';
    keyboardNav: '100% keyboard accessible';
    screenReader: 'Screen reader compatible';
  };
  
  security: {
    vulnerabilities: 'No high/critical vulnerabilities';
    authentication: 'All auth flows secured';
    dataProtection: 'PII properly protected';
  };
}
```

This comprehensive testing strategy ensures that the CleanStation digitalization system meets all quality, performance, security, and accessibility requirements while maintaining reliable operation in the production environment. 