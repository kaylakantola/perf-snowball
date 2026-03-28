# Local Benchmarking Reference

Methods for proving performance improvements locally before merging. Each fix should include a local benchmark that demonstrates the improvement independent of production data.

## Choosing the Right Method

Match the benchmarking method to the fix type:

| Fix Type | Recommended Method | What to Measure |
|----------|-------------------|-----------------|
| N+1 query / eager loading | Rails console + ActiveSupport::Notifications | Query count + total query time |
| Slow database query | EXPLAIN ANALYZE + Benchmark.measure | Execution plan + wall clock time |
| Endpoint latency | rack-mini-profiler or Benchmark.measure | Request duration + breakdown |
| Frontend page load | Lighthouse CI | LCP, TTI, CLS, total bundle size |
| React render performance | React Profiler | Component render count + duration |
| Memory usage | memory_profiler gem / Chrome DevTools | Memory allocation before/after |

## Backend (Rails)

### Counting and Timing SQL Queries

Use ActiveSupport::Notifications to count queries in a block:

```ruby
queries = []
callback = lambda { |_name, start, finish, _id, payload|
  queries << { sql: payload[:sql], duration: (finish - start) * 1000 }
}
ActiveSupport::Notifications.subscribed(callback, "sql.active_record") do
  # Run the code being benchmarked
  result = SomeController.new.some_action_query
end
puts "Query count: #{queries.size}"
puts "Total query time: #{queries.sum { |q| q[:duration] }.round(2)}ms"
queries.each { |q| puts "  #{q[:duration].round(2)}ms: #{q[:sql][0..100]}" }
```

Run this before and after the fix to show the difference.

### Timing with Benchmark

```ruby
require 'benchmark'

# Run multiple iterations for stability
n = 10
Benchmark.bm(15) do |x|
  x.report("before fix:") { n.times { original_code_path } }
  x.report("after fix:")  { n.times { new_code_path } }
end
```

### EXPLAIN ANALYZE for Query Changes

For MySQL:
```sql
EXPLAIN ANALYZE SELECT ... ;  -- the original query
EXPLAIN ANALYZE SELECT ... ;  -- the optimized query
```

Key things to compare:
- Rows examined vs rows returned
- Index usage (should see "Using index" after fix)
- Execution time

### rack-mini-profiler

If installed, access any page with `?pp=flamegraph` to get detailed timing breakdowns. Capture the output before and after.

## Frontend (React)

### Lighthouse CI

Run Lighthouse against a local dev server:

```bash
# Install if needed
npm install -g lighthouse

# Run against local server
lighthouse http://localhost:3000/page-to-test --output json --output-path evidence/fix-XXX-lighthouse-before.json

# After fix
lighthouse http://localhost:3000/page-to-test --output json --output-path evidence/fix-XXX-lighthouse-after.json
```

Key metrics to compare: LCP, TTI, CLS, Total Blocking Time, Speed Index.

### React Profiler

Wrap the component being optimized with React Profiler:

```jsx
<Profiler id="ComponentName" onRender={(id, phase, actualDuration) => {
  console.log(`${id} ${phase}: ${actualDuration.toFixed(2)}ms`);
}}>
  <ComponentBeingOptimized />
</Profiler>
```

Record render counts and durations before/after.

### Bundle Size

```bash
# Before fix
npx webpack --json > evidence/fix-XXX-bundle-before.json

# After fix
npx webpack --json > evidence/fix-XXX-bundle-after.json
```

Or use `source-map-explorer` for visual diff.

## Capturing Results

After running benchmarks, save results to `~/.claude/snowball-performance-log/evidence/fix-[number]-local-benchmark.md` with:

1. **Test method** -- which tool/technique was used
2. **Before results** -- exact commands and output
3. **After results** -- exact commands and output
4. **Reproduction steps** -- how anyone can re-run this benchmark

Include these results in the PR description under the "Local Benchmark Results" section.
