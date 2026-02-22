</head>
<body>

<h1>Designing a Scalable URL Shortener (like Bitly)</h1>

<h2>1. Requirements</h2>

<h3>Functional Requirements</h3>
<ul>
    <li>User submits long URL → gets short URL</li>
    <li>Redirect short URL → original URL</li>
    <li>Custom alias support (optional)</li>
    <li>Expiration time (optional)</li>
    <li>Analytics (click count, timestamp, IP, geo)</li>
</ul>

<h3>Non-Functional Requirements</h3>
<ul>
    <li>High availability</li>
    <li>Low latency redirects (&lt;100ms)</li>
    <li>Horizontally scalable</li>
    <li>Highly durable storage</li>
    <li>Handle millions of URLs/day</li>
</ul>

<h2>2. High-Level Design</h2>

<h3>Core Components</h3>
<ul>
    <li><strong>API Gateway</strong>
        <ul>
            <li>Rate limiting</li>
            <li>Auth</li>
            <li>Routing</li>
        </ul>
    </li>
    <li><strong>URL Service</strong>
        <ul>
            <li>Generates short IDs</li>
            <li>Stores mapping</li>
            <li>Handles redirection</li>
        </ul>
    </li>
    <li><strong>Database</strong>
        <ul>
            <li>Stores URL mapping</li>
        </ul>
    </li>
    <li><strong>Cache (Redis)</strong>
        <ul>
            <li>Stores frequently accessed URLs</li>
        </ul>
    </li>
    <li><strong>Analytics Pipeline</strong>
        <ul>
            <li>Event streaming (Kafka)</li>
            <li>Aggregation worker</li>
        </ul>
    </li>
</ul>

<h3>Architecture Diagram (Conceptual)</h3>
<pre>
Client
  ↓
Load Balancer
  ↓
API Service
  ↓
Redis Cache → If miss → Database
  ↓
Redirect

Analytics events → Kafka → Worker → Analytics DB
</pre>

<h2>3. Database Schema</h2>

<h3>Table: urls</h3>
<table>
    <tr>
        <th>Field</th>
        <th>Type</th>
        <th>Notes</th>
    </tr>
    <tr>
        <td>id</td>
        <td>bigint</td>
        <td>Primary Key</td>
    </tr>
    <tr>
        <td>short_code</td>
        <td>varchar(10)</td>
        <td>Indexed, unique</td>
    </tr>
    <tr>
        <td>long_url</td>
        <td>text</td>
        <td></td>
    </tr>
    <tr>
        <td>created_at</td>
        <td>timestamp</td>
        <td></td>
    </tr>
    <tr>
        <td>expires_at</td>
        <td>timestamp</td>
        <td>Nullable</td>
    </tr>
    <tr>
        <td>user_id</td>
        <td>bigint</td>
        <td>Optional</td>
    </tr>
</table>

<h3>Table: click_events</h3>
<table>
    <tr>
        <th>Field</th>
        <th>Type</th>
    </tr>
    <tr>
        <td>id</td>
        <td>bigint</td>
    </tr>
    <tr>
        <td>short_code</td>
        <td>varchar(10)</td>
    </tr>
    <tr>
        <td>timestamp</td>
        <td>timestamp</td>
    </tr>
    <tr>
        <td>ip_address</td>
        <td>varchar(45)</td>
    </tr>
    <tr>
        <td>country</td>
        <td>varchar(50)</td>
    </tr>
</table>

<h4>Indexes</h4>
<ul>
    <li>Index on <code>short_code</code></li>
    <li>Index on <code>timestamp</code></li>
</ul>

<h2>4. ID Generation Strategy</h2>

<h3>Options</h3>

<h4>Option 1: Auto-increment + Base62 encoding</h4>
<ul>
    <li>DB auto ID → encode to Base62</li>
    <li>Simple</li>
    <li>Predictable sequence (minor security concern)</li>
</ul>

<h4>Option 2: Snowflake IDs</h4>
<ul>
    <li>Distributed unique ID</li>
    <li>No DB bottleneck</li>
    <li>Good for horizontal scaling</li>
</ul>

<h4>Option 3: Random string</h4>
<ul>
    <li>Collision handling required</li>
</ul>

<p><strong>Recommended:</strong> Snowflake or distributed ID service</p>

<h2>5. Bottlenecks</h2>

<h3>1. Database Write Contention</h3>
<ul>
    <li>Heavy write load</li>
    <li>Solution: Sharding</li>
</ul>

<h3>2. Hot URLs</h3>
<ul>
    <li>Viral links overload DB</li>
    <li>Solution: Redis caching</li>
</ul>

<h3>3. Analytics Write Volume</h3>
<ul>
    <li>Millions of click events</li>
    <li>Solution: Kafka buffer + async workers</li>
</ul>

<h2>6. Scaling Strategy</h2>

<h3>Phase 1: Vertical Scaling</h3>
<ul>
    <li>Single DB instance</li>
    <li>Redis cache</li>
</ul>

<h3>Phase 2: Read Replicas</h3>
<ul>
    <li>Redirect reads → replicas</li>
</ul>

<h3>Phase 3: Sharding</h3>
<ul>
    <li>Shard by short_code hash</li>
</ul>

<h3>Phase 4: Global Scaling</h3>
<ul>
    <li>Geo-distributed CDN</li>
    <li>Regional DB clusters</li>
</ul>

<h2>7. Reliability</h2>
<ul>
    <li>Use Redis TTL for cache invalidation</li>
    <li>Health checks on all services</li>
    <li>Circuit breaker pattern</li>
    <li>Retry with exponential backoff</li>
</ul>

<h2>8. Security Considerations</h2>
<ul>
    <li>Prevent malicious URLs</li>
    <li>Rate limiting</li>
    <li>Abuse detection</li>
    <li>HTTPS only</li>
    <li>Signed URLs for private links</li>
</ul>

</body>
</html>
