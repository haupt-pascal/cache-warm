# 🔥 CACHE WARMER CONCEPT 🔥

## 📋 Overview

This document describes a concept for a cache warmer specifically designed for Varnish and Shopware. It uses Bun and TypeScript for maximum performance and efficiency.

## 🎯 Goals

- **Speed Focus** 🏎️ Fast warming of Varnish and Shopware caches
- **Completeness** 📊 Process all URLs from the sitemap
- **Smart Crawling** 🧠 Intelligent prioritization of important pages
- **Resource Efficiency** 🌱 Minimal server load
- **Monitoring** 👀 Live monitoring of cache status
- **Validation** ✅ Verification of cache effectiveness
- **Reporting** 📨 Comprehensive email reporting

## 🛠️ Tech Stack

- **Bun** 🥟 - Ultra-fast JavaScript/TypeScript runtime
- **TypeScript** 📝 - For type-safety and better developer experience
- **Puppeteer/Playwright** 🎭 - For headless browser interactions (scrolling)
- **Zod** ✅ - For validation
- **Axios** 🔄 - For simple HTTP requests
- **pino** 📊 - For structured logging
- **cron** ⏰ - For scheduled execution
- **Nodemailer** 📧 - For email reporting

## 🏗️ Architecture

### Components

// Example of email reporting configuration
interface EmailConfig {
  enabled: boolean;
  recipients: string[];
  sender: string;
  smtpConfig: {
    host: string;
    port: number;
    secure: boolean;
    auth: {
      user: string;
      pass: string;
    }
  };
  thresholds: {
    failureRate: number;
    cacheHitRate: number;
    avgLoadTime: number;
  };
  templates: {
    success: string;
    warning: string;
    error: string;
  };
  includeAttachments: boolean;
}
```

Example configuration:

```json
{
  "email": {
    "enabled": true,
    "recipients": ["team@example.com", "admin@example.com"],
    "sender": "cache-warmer@example.com",
    "smtpConfig": {
      "host": "smtp.example.com",
      "port": 587,
      "secure": false,
      "auth": {
        "user": "reporting@example.com",
        "pass": "password"
      }
    },
    "thresholds": {
      "failureRate": 0.1,
      "cacheHitRate": 0.9,
      "avgLoadTime": 2000
    },
    "templates": {
      "success": "email-templates/success.hbs",
      "warning": "email-templates/warning.hbs",
      "error": "email-templates/error.hbs"
    },
    "includeAttachments": true
  }
}
```

- Easily configurable email reporting
- Template-based emails with customizable designs
- Threshold-based alerting
- Support for multiple recipients
- HTML and text formats with optional attachments

## 💼 Benefits

- **Better User Experience** 👍 through pre-warmed caches
- **Lower Server Load** 📉 during peak times
- **Higher Conversion Rates** 💰 through faster load times
- **Better SEO Ranking** 🔍 through improved performance metrics
- **Proactive Issue Detection** 🚨 through validation and monitoring

## 🌐 Scaling Considerations

### Multi-Server Deployment

The cache warmer can be deployed across multiple servers or containers for increased performance. The architecture supports distributed execution with:

- Central configuration
- Shared results storage
- Work distribution mechanisms
- Load balancing between nodes

### Cloud Integration

For cloud environments, the system can be integrated with:

- AWS Lambda for serverless execution
- Docker containers for containerized deployment
- Azure Functions or Google Cloud Functions
- CI/CD pipelines for automated execution

### Performance Optimization

For large-scale deployments:

- URL batching and prioritization
- Memory usage optimization
- Connection pooling
- Streaming results processing

## 📝 Conclusion

This cache warmer provides an efficient solution for pre-warming Varnish and Shopware caches. By using Bun and TypeScript, maximum performance is achieved, while the modular architecture facilitates extensions and customizations.

The modular design allows the system to be easily adapted to different environments and scaled up for larger deployments. With comprehensive validation and reporting features, it not only warms caches but also provides valuable insights into performance and potential issues.

---

*Yeet your cache problems away with this fire solution!* 🔥👾
cache-warmer/
├── src/
│   ├── index.ts         # Main entry point
│   ├── config.ts        # Configuration management
│   ├── sitemap/         # Sitemap processing
│   │   ├── parser.ts    # XML parser for sitemaps
│   │   └── fetcher.ts   # Loads sitemap files
│   ├── crawler/         # Crawler logic
│   │   ├── browser.ts   # Browser control
│   │   ├── queue.ts     # URL queue management
│   │   └── worker.ts    # Worker for parallel processing
│   ├── validator/       # Cache validation
│   │   ├── checker.ts   # Validates cache effectiveness
│   │   └── rules.ts     # Validation rules
│   ├── metrics/         # Metrics and monitoring
│   │   ├── collector.ts # Collects performance data
│   │   └── reporter.ts  # Reports results
│   ├── reporting/       # Reporting functionality
│   │   ├── email.ts     # Email reporting
│   │   ├── templates/   # Email templates
│   │   └── formatter.ts # Report formatting
│   └── utils/           # Helper functions
├── plugins/             # Modular extensions
│   ├── shopware.ts      # Shopware-specific functionality
│   └── varnish.ts       # Varnish-specific functionality
├── config/              # Configuration files
│   └── default.json     # Default configuration
├── package.json         # Dependencies
└── tsconfig.json        # TypeScript configuration
```

## 💡 How It Works

### 1. Sitemap Processing 🗺️

```typescript
// Example of the sitemap parser
export async function parseSitemap(url: string): Promise<URL[]> {
  const response = await fetch(url);
  const xml = await response.text();
  
  // Parse XML and extract URLs
  const urls: URL[] = [];
  // XML parsing logic here...
  
  return urls;
}
```

- Supports multiple sitemap formats
- Recursive processing of sitemap indices
- Prioritization based on `<priority>` and `<lastmod>` tags
- Filters for specific URL patterns

### 2. Crawling Engine 🕸️

```typescript
// Example of a worker
export async function crawlPage(url: URL): Promise<CrawlResult> {
  const browser = await launchBrowser();
  const page = await browser.newPage();
  
  const startTime = performance.now();
  await page.goto(url.toString(), { waitUntil: 'networkidle2' });
  
  // Scroll simulation for complete loading
  await autoScroll(page);
  
  const loadTime = performance.now() - startTime;
  
  await browser.close();
  return { url, loadTime, success: true };
}

async function autoScroll(page: Page): Promise<void> {
  await page.evaluate(async () => {
    await new Promise<void>((resolve) => {
      let totalHeight = 0;
      const distance = 100;
      const timer = setInterval(() => {
        const scrollHeight = document.body.scrollHeight;
        window.scrollBy(0, distance);
        totalHeight += distance;
        
        if (totalHeight >= scrollHeight) {
          clearInterval(timer);
          resolve();
        }
      }, 100);
    });
  });
}
```

- Parallel processing with configurable concurrency limit
- Automatic scrolling to simulate user interactions
- Intelligent retry management for errors
- Device and viewport simulation options

### 3. Configuration Management ⚙️

```typescript
// Example of a configuration interface
interface CacheWarmerConfig {
  concurrency: number;
  userAgent: string;
  sitemapUrl: string;
  ignorePatterns: string[];
  verbose: boolean;
  timeout: number;
  browser: {
    headless: boolean;
    args: string[];
  };
  email: {
    enabled: boolean;
    recipients: string[];
    sender: string;
    smtpConfig: SMTPConfig;
  };
  plugins: string[];
}

// Loading and validating the configuration
const config: CacheWarmerConfig = loadConfig();
```

- Flexible configuration via environment variables and configuration files
- Validation using Zod
- Defaults for easy setup
- Plugin system for extensibility

### 4. Cache Validation System ✅

```typescript
// Example of cache validation
export async function validateCache(url: string): Promise<ValidationResult> {
  // First request - should be a cache miss
  const firstResponse = await fetch(url, {
    method: 'GET',
    headers: {
      'User-Agent': config.userAgent,
      'Cache-Control': 'no-cache',
    },
  });
  
  const firstCacheHeader = firstResponse.headers.get('X-Cache') || '';
  
  // Second request - should be a cache hit
  const secondResponse = await fetch(url, {
    method: 'GET',
    headers: {
      'User-Agent': config.userAgent,
    },
  });
  
  const secondCacheHeader = secondResponse.headers.get('X-Cache') || '';
  
  return {
    url,
    firstRequest: {
      status: firstResponse.status,
      cacheHeader: firstCacheHeader,
      isCached: firstCacheHeader.includes('HIT'),
    },
    secondRequest: {
      status: secondResponse.status,
      cacheHeader: secondCacheHeader,
      isCached: secondCacheHeader.includes('HIT'),
    },
    isValid: !firstCacheHeader.includes('HIT') && secondCacheHeader.includes('HIT'),
  };
}
```

- Detailed cache header analysis
- Support for Varnish-specific headers
- Shopware cache validation
- Custom validation rules through plugins

### 5. Metrics and Monitoring 📈

```typescript
// Example of a metrics collector
export class MetricsCollector {
  private results: CrawlResult[] = [];
  
  addResult(result: CrawlResult): void {
    this.results.push(result);
  }
  
  generateReport(): Report {
    const totalUrls = this.results.length;
    const successfulUrls = this.results.filter(r => r.success).length;
    const avgLoadTime = this.results.reduce((sum, r) => sum + r.loadTime, 0) / totalUrls;
    const cacheHitRate = this.results.filter(r => r.cacheValidation?.isValid).length / totalUrls;
    
    return {
      totalUrls,
      successfulUrls,
      failedUrls: totalUrls - successfulUrls,
      avgLoadTime,
      cacheHitRate,
      slowestPages: this.getSlowestPages(5),
      mostProblematicPages: this.getMostProblematicPages(5),
      startTime: this.startTime,
      endTime: new Date(),
    };
  }
  
  private getSlowestPages(count: number): PageMetric[] {
    return [...this.results]
      .sort((a, b) => b.loadTime - a.loadTime)
      .slice(0, count)
      .map(r => ({ url: r.url.toString(), loadTime: r.loadTime }));
  }
  
  private getMostProblematicPages(count: number): PageMetric[] {
    return [...this.results]
      .filter(r => !r.success || !r.cacheValidation?.isValid)
      .slice(0, count)
      .map(r => ({ 
        url: r.url.toString(), 
        issue: !r.success ? 'Load Failed' : 'Cache Validation Failed' 
      }));
  }
}
```

- Detailed logs for debugging and analysis
- Performance metrics (load times, cache hits, etc.)
- Optional: Integration with monitoring systems (Prometheus, etc.)
- Historical data tracking for trend analysis

### 6. Email Reporting System 📧

```typescript
// Example of email reporting
export async function sendEmailReport(report: Report): Promise<void> {
  if (!config.email.enabled) return;
  
  const transporter = nodemailer.createTransport(config.email.smtpConfig);
  
  const htmlReport = generateHtmlReport(report);
  
  await transporter.sendMail({
    from: config.email.sender,
    to: config.email.recipients.join(', '),
    subject: `Cache Warming Report - ${new Date().toLocaleDateString()}`,
    html: htmlReport,
    attachments: [
      {
        filename: 'cache-report.json',
        content: JSON.stringify(report, null, 2),
      },
    ],
  });
}

function generateHtmlReport(report: Report): string {
  // Generate HTML from template
  return emailTemplate({
    ...report,
    executionTime: formatDuration(report.endTime.getTime() - report.startTime.getTime()),
    successRate: `${((report.successfulUrls / report.totalUrls) * 100).toFixed(2)}%`,
    cacheHitRate: `${(report.cacheHitRate * 100).toFixed(2)}%`,
  });
}
```

- Configurable email templates
- HTML and plain text formats
- Attachment options for detailed reports
- Threshold-based notifications

## 🚀 Implementation Plan

### Phase 1: Core Functionality

1. Project setup with Bun and TypeScript
2. Implementation of the sitemap parser
3. Basic crawling with foundational functionality
4. Core configuration options
5. Simple cache validation

### Phase 2: Extensions

1. Parallel crawling and queue management
2. Scrolling and advanced browser interactions
3. Enhanced metrics and reporting
4. Email reporting system implementation
5. Error handling and retries

### Phase 3: Optimization

1. Performance optimizations
2. Resource management for minimal server load
3. Intelligent URL prioritization
4. Advanced cache status validation
5. Plugin architecture for extensibility

## 🤔 Advanced Concepts

### Modular Plugin System

```typescript
// Example of plugin interface
export interface CacheWarmerPlugin {
  name: string;
  init(config: PluginConfig): Promise<void>;
  beforeCrawl?(url: URL): Promise<URL | null>;
  afterCrawl?(result: CrawlResult): Promise<CrawlResult>;
  validateCache?(url: URL): Promise<ValidationResult>;
  enhanceReport?(report: Report): Promise<Report>;
}

// Example plugin manager
export class PluginManager {
  private plugins: CacheWarmerPlugin[] = [];
  
  async loadPlugins(pluginNames: string[]): Promise<void> {
    for (const name of pluginNames) {
      try {
        const pluginModule = await import(`../plugins/${name}`);
        const plugin = new pluginModule.default();
        await plugin.init(this.config.plugins[name] || {});
        this.plugins.push(plugin);
      } catch (error) {
        console.error(`Failed to load plugin: ${name}`, error);
      }
    }
  }
  
  async runHook<T>(hookName: string, data: T): Promise<T> {
    let result = data;
    for (const plugin of this.plugins) {
      if (typeof plugin[hookName] === 'function') {
        result = await plugin[hookName](result);
        if (result === null) break;
      }
    }
    return result;
  }
}
```

- Extensible plugin system for custom functionality
- Hook-based architecture
- Configuration per plugin
- Support for Shopware and Varnish specific extensions

### Differential Crawling

```typescript
// Example of differential crawling
async function shouldCrawl(url: URL, lastCrawled: Map<string, number>): Promise<boolean> {
  const urlString = url.toString();
  const lastCrawl = lastCrawled.get(urlString);
  
  if (!lastCrawl) return true;
  
  // Check if the URL was recently crawled
  const now = Date.now();
  const hoursSinceLastCrawl = (now - lastCrawl) / (1000 * 60 * 60);
  
  // Priority-based decision
  if (isHighPriorityPage(url)) return hoursSinceLastCrawl >= 1;
  if (isMediumPriorityPage(url)) return hoursSinceLastCrawl >= 6;
  return hoursSinceLastCrawl >= 24;
}
```

- Intelligent decision making for which URLs need to be re-crawled
- Prioritization based on page type and importance
- Historical data to optimize crawl frequency
- Integration with cache validation results

### Distributed Architecture

```typescript
// Example controller for distributed crawling
export class DistributedController {
  private workers: Worker[] = [];
  private queue: Queue;
  
  constructor(config: DistributedConfig) {
    this.queue = new Queue();
    
    // Initialize workers
    for (let i = 0; i < config.workerCount; i++) {
      const worker = new Worker(config.workerScript);
      worker.on('message', this.handleWorkerMessage.bind(this));
      this.workers.push(worker);
    }
  }
  
  async start(urls: URL[]): Promise<void> {
    // Add all URLs to the queue
    for (const url of urls) {
      this.queue.add(url);
    }
    
    // Distribute initial work
    this.distributeWork();
  }
  
  private distributeWork(): void {
    for (const worker of this.workers) {
      if (worker.idle && !this.queue.isEmpty()) {
        const url = this.queue.next();
        worker.postMessage({ type: 'crawl', url: url.toString() });
        worker.idle = false;
      }
    }
  }
  
  private handleWorkerMessage(message: WorkerMessage): void {
    if (message.type === 'result') {
      this.collector.addResult(message.result);
      this.workers[message.workerId].idle = true;
      this.distributeWork();
    }
  }
}
```

- Scalable architecture for large-scale crawling
- Worker-based distribution of tasks
- Load balancing between workers
- Support for multi-node deployment

## 🧪 Testing

```typescript
// Example of a sitemap parser test
import { parseSitemap } from '../src/sitemap/parser';

describe('Sitemap Parser', () => {
  it('should parse valid sitemap XML', async () => {
    const mockXml = `<?xml version="1.0" encoding="UTF-8"?>
    <urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
      <url>
        <loc>https://example.com/</loc>
        <lastmod>2023-01-01</lastmod>
        <priority>1.0</priority>
      </url>
    </urlset>`;
    
    // Mock fetch implementation
    global.fetch = jest.fn().mockResolvedValue({
      text: () => Promise.resolve(mockXml),
    });
    
    const urls = await parseSitemap('https://example.com/sitemap.xml');
    expect(urls).toHaveLength(1);
    expect(urls[0].toString()).toBe('https://example.com/');
  });
});
```

- Unit tests for core functionality
- Integration tests for the entire workflow
- Performance tests for optimizations
- Mocking for external dependencies

## 🔄 Scheduled Execution

```typescript
// Example of cron setup
import { CronJob } from 'cron';
import { startCacheCrawl } from './crawler';

// Run daily at 3 AM
const job = new CronJob('0 3 * * *', async () => {
  console.log('🚀 Starting scheduled cache warm-up');
  await startCacheCrawl(config);
  console.log('✅ Cache warm-up completed');
});

job.start();
```

- Configurable schedules for regular execution
- Event-based triggers (e.g., after deployments)
- Rate-limiting for server protection
- Support for staggered execution patterns

## 📧 Email Reporting Configuration

```typescript
// Example of email reporting configuration
interface EmailConfig {
  enabled: boolean;
  recipients: string[];
  sender: string;
  smtpConfig:
```
---

*Yeet your cache problems away with this fire solution!* 🔥👾
