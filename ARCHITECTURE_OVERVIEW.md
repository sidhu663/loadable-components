# Loadable Components - Architecture Overview

## Project Overview

Loadable Components is a React code splitting library that provides a comprehensive solution for dynamic imports with server-side rendering (SSR) support. It's the recommended code splitting library by the React team and serves as a production-ready alternative to React.lazy with enhanced SSR capabilities.

## Repository Structure

This is a monorepo managed by **Yarn workspaces** and **Lerna**, containing multiple interconnected packages:

```
loadable-components/
├── packages/
│   ├── component/          # Core React component library
│   ├── server/             # Server-side rendering utilities
│   ├── babel-plugin/       # Babel transformation plugin
│   ├── webpack-plugin/     # Webpack integration plugin
│   └── codemod/           # Code transformation utilities
├── examples/              # Usage examples and fixtures
├── website/              # Documentation website
└── scripts/              # Build and release scripts
```

## Core Architecture

### 1. Core Package: @loadable/component

**Location**: `packages/component/`
**Main Entry**: `src/loadable.js`
**Key Files**:
- `createLoadable.js` - Core loadable component factory
- `loadableReady.js` - Client-side readiness utility
- `resolvers.js` - Component resolution logic
- `Context.js` - React context for chunk extraction

#### Key Abstractions:

**createLoadable Factory**:
```javascript
function createLoadable({
  defaultResolveComponent = identity,
  render,
  onLoad,
})
```

The factory creates two main functions:
- `loadable(loadableConstructor, options)` - Creates loadable components
- `lazy(ctor, options)` - Creates Suspense-compatible components

**InnerLoadable Component**:
- Manages component loading state (PENDING, RESOLVED, REJECTED)
- Handles synchronous and asynchronous loading
- Implements caching mechanism
- Provides lifecycle hooks for loading events

**Component States**:
- `STATUS_PENDING` - Component is being loaded
- `STATUS_RESOLVED` - Component loaded successfully  
- `STATUS_REJECTED` - Component loading failed

### 2. Server Package: @loadable/server

**Location**: `packages/server/`
**Main Entry**: `src/index.js`
**Key Files**:
- `ChunkExtractor.js` - Core server-side chunk collection
- `ChunkExtractorManager.js` - React context provider
- `util.js` - Server utilities

#### Key Abstractions:

**ChunkExtractor Class**:
```javascript
class ChunkExtractor {
  constructor({
    statsFile,
    stats,
    entrypoints = ['main'],
    namespace = '',
    outputPath,
    publicPath,
    inputFileSystem = fs,
  })
}
```

**Core Methods**:
- `collectChunks(app)` - Wraps React app to collect chunks
- `getScriptTags()` - Returns HTML script tags
- `getScriptElements()` - Returns React script elements
- `getStyleTags()` - Returns CSS link tags
- `getLinkTags()` - Returns preload/prefetch links

### 3. Babel Plugin: @loadable/babel-plugin

**Location**: `packages/babel-plugin/`
**Purpose**: 
- Transforms dynamic imports for SSR compatibility
- Generates chunk names automatically
- Adds `requireSync` and `resolve` methods to loadable calls

**Required for**: Server-side rendering and automatic chunk naming

### 4. Webpack Plugin: @loadable/webpack-plugin

**Location**: `packages/webpack-plugin/`
**Purpose**:
- Generates `loadable-stats.json` file
- Provides chunk metadata for server-side rendering
- Integrates with Webpack's compilation process

## Data Flow Architecture

### Client-Side Flow

1. **Component Definition**:
   ```javascript
   const MyComponent = loadable(() => import('./MyComponent'))
   ```

2. **Component Rendering**:
   - Initial render shows fallback (loading state)
   - Dynamic import triggered asynchronously
   - Component resolves and re-renders with actual content

3. **Caching**:
   - Resolved components cached by cache key
   - Subsequent renders use cached version
   - Cache invalidation on prop changes (if cacheKey provided)

### Server-Side Flow

1. **Build Time**:
   - Babel plugin transforms loadable calls
   - Webpack plugin generates stats file
   - Chunks and dependencies mapped

2. **Server Rendering**:
   ```javascript
   const extractor = new ChunkExtractor({ statsFile })
   const jsx = extractor.collectChunks(<App />)
   const html = renderToString(jsx)
   const scriptTags = extractor.getScriptTags()
   ```

3. **Chunk Collection**:
   - ChunkExtractorManager provides context
   - Loadable components register chunks during render
   - Extractor collects all required chunks

4. **Client Hydration**:
   ```javascript
   loadableReady(() => {
     hydrate(<App />, document.getElementById('root'))
   })
   ```

## Key Design Patterns

### 1. Factory Pattern
- `createLoadable` factory creates customized loadable functions
- Allows different rendering strategies and component resolution

### 2. Provider Pattern
- `ChunkExtractorManager` provides chunk collection context
- `Context.Consumer` pattern for accessing extractor

### 3. Higher-Order Component Pattern
- `withChunkExtractor` HOC injects chunk extractor
- Loadable components wrapped with forwarded refs

### 4. Caching Strategy
- Component-level caching with configurable cache keys
- Promise-based caching for async operations
- Cache invalidation on key changes

### 5. State Machine Pattern
- Loading states: PENDING → RESOLVED/REJECTED
- State transitions managed by component lifecycle

## Integration Points

### Webpack Integration
- Stats file generation via `@loadable/webpack-plugin`
- Chunk metadata extraction
- Asset path resolution

### Babel Integration
- AST transformation of dynamic imports
- Automatic chunk name generation
- SSR compatibility transformations

### React Integration
- Component lifecycle management
- Context API for chunk collection
- Suspense compatibility (lazy function)

### Server Framework Integration
- Express.js compatible
- Streaming rendering support
- Custom server implementations

## Performance Optimizations

### 1. Prefetching
- Automatic prefetch link generation
- Resource hints for better loading performance
- Configurable prefetch strategies

### 2. Code Splitting Strategies
- Route-based splitting
- Component-based splitting
- Library splitting support

### 3. Bundle Optimization
- Tree shaking support
- Minimal runtime overhead
- Efficient chunk loading

### 4. Caching
- Component-level caching
- Browser cache utilization
- CDN-friendly asset URLs

## Error Handling

### Client-Side
- Graceful fallback on loading errors
- Error boundaries integration
- Retry mechanisms

### Server-Side
- Missing chunk detection
- Fallback rendering strategies
- Development vs production error handling

## Development vs Production

### Development Mode
- Enhanced error messages
- Hot module replacement support
- Development-only validations

### Production Mode
- Optimized bundle sizes
- Minimal runtime overhead
- Production error handling

## Extension Points

### Custom Resolvers
```javascript
const customLoadable = createLoadable({
  defaultResolveComponent: (module) => module.MyComponent,
  render: ({ result: Component, props }) => <Component {...props} />,
})
```

### Custom Fallbacks
- Component-level fallbacks
- Global fallback configuration
- Suspense integration

### Custom Cache Keys
- Dynamic cache key generation
- Prop-based cache invalidation
- Custom caching strategies

## Dependencies

### Core Dependencies
- `react` - Core React library
- `hoist-non-react-statics` - Static property hoisting
- `react-is` - React element type checking

### Build Dependencies
- `@babel/runtime` - Babel runtime helpers
- `lodash` - Utility functions (server package)
- Various Babel and Rollup plugins for building

## Browser Support

- Modern browsers with ES2015+ support
- Node.js 8+ for server-side rendering
- Webpack 4+ for build integration

## Security Considerations

- Subresource Integrity (SRI) support
- Content Security Policy (CSP) compatibility
- Secure asset loading practices

## Testing Infrastructure

### Testing Framework

Loadable Components uses **Jest** as its primary testing framework with the following setup:

- **Jest v24.9.0** - Core testing framework
- **@testing-library/react** - React component testing utilities
- **@testing-library/jest-dom** - Custom Jest matchers for DOM assertions
- **babel-jest** - Babel integration for ES6+ and JSX transpilation
- **regenerator-runtime** - Async/await support in tests

### Test Structure

Tests are co-located with source files using the `.test.js` naming convention:

```
packages/
├── component/src/
│   ├── loadable.js
│   └── loadable.test.js          # Component tests
├── server/src/
│   ├── ChunkExtractor.js
│   ├── ChunkExtractor.test.js    # Server tests
│   ├── util.js
│   └── util.test.js              # Utility tests
└── babel-plugin/src/
    ├── index.js
    └── index.test.js             # Babel plugin tests
```

### Test Categories

#### 1. Component Tests (`loadable.test.js`)

**Core Functionality Tests**:
- Component loading states (pending, resolved, rejected)
- Fallback rendering behavior
- Error boundary integration
- Suspense compatibility
- Preloading functionality
- Cache key behavior
- Forward ref support

**Example Test Patterns**:
```javascript
it('mounts component when loaded', async () => {
  const load = resolvedToDefault(() => 'loaded')
  const Component = loadable(load)
  const { container } = render(<Component />)
  expect(container).toBeEmpty()
  await wait(() => expect(container).toHaveTextContent('loaded'))
})

it('supports preload', async () => {
  const load = resolvedToDefault(() => 'loaded')
  const Component = loadable(load)
  Component.preload({ foo: 'bar' })
  expect(load).toHaveBeenCalledWith({ foo: 'bar' })
})
```

#### 2. Server Tests (`ChunkExtractor.test.js`)

**ChunkExtractor Functionality**:
- Script tag generation
- Style tag generation
- Link tag generation (preload/prefetch)
- Chunk collection and dependencies
- Asset integrity support
- Namespace support
- Extra props handling

**Example Test Patterns**:
```javascript
it('should return main script tag without chunk', () => {
  expect(extractor.getScriptTags()).toMatchInlineSnapshot(`
    "<script id=\"__LOADABLE_REQUIRED_CHUNKS__\" type=\"application/json\">[]</script>
    <script async data-chunk=\"main\" src=\"/dist/node/main.js\"></script>"
  `)
})
```

#### 3. Babel Plugin Tests (`index.test.js`)

**AST Transformation Tests**:
- Dynamic import transformations
- Chunk name generation
- SSR compatibility transformations
- Error handling for invalid syntax

### Test Utilities and Helpers

#### Mock Functions
```javascript
// Unresolvable promise for testing loading states
const unresolvableLoad = jest.fn(() => new Promise(() => {}))

// Resolved promise helper
const resolvedToDefault = value =>
  jest.fn().mockResolvedValue({ default: value })

// Delayed resolution for timing tests
function mockDelayedResolvedValueOnce(fn, resolvedValue) {
  return fn.mockImplementationOnce(
    () => new Promise(resolve => {
      setTimeout(() => resolve(resolvedValue), 1000)
    })
  )
}
```

#### Error Boundary Test Component
```javascript
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props)
    this.state = { error: false, retries: props.retries || 0 }
  }

  componentDidCatch() {
    this.setState(prevState => ({
      error: true,
      retries: prevState.retries - 1,
    }))
  }

  render() {
    const { children, fallback } = this.props
    const { error, retries } = this.state
    
    if (error) {
      return (retries >= 0 && children) || fallback || null
    }
    return children || null
  }
}
```

### Test Fixtures and Examples

#### Fixture Generation
The project uses a sophisticated fixture system:

- **`scripts/prepare.sh`** - Builds examples and generates test fixtures
- **`examples/server-side-rendering`** - Full SSR example for integration testing
- **`packages/server/__fixtures__/stats.json`** - Webpack stats fixture for server tests

#### Example Projects for Testing
```
examples/
├── client-side/              # Client-only code splitting
├── server-side-rendering/    # Full SSR implementation
├── server-side-rendering-async-node/ # Async Node.js SSR
├── suspense/                 # React Suspense integration
├── typescript/               # TypeScript usage
├── razzle/                   # Razzle framework integration
└── webpack/                  # Webpack configuration examples
```

### Running Tests

#### Development Workflow
```bash
# Install dependencies
yarn install

# Build packages
yarn build

# Prepare test fixtures
yarn test:prepare

# Run all tests
yarn test

# Run tests in CI mode
yarn test --ci

# Development mode with watch
yarn dev  # Builds packages in watch mode
```

#### Test Commands
- `yarn test:prepare` - Builds examples and generates fixtures
- `yarn test` - Runs Jest test suite
- `yarn test --ci` - Runs tests in CI mode with coverage
- `yarn lint` - Runs ESLint for code quality

## Contributing to Loadable Components

### Development Setup

#### Prerequisites
- **Node.js 8+** - Runtime environment
- **Yarn** - Package manager (required, not npm)
- **Git** - Version control

#### Setup Process
```bash
# 1. Fork and clone the repository
git clone https://github.com/<your_username>/loadable-components
cd loadable-components
git checkout -b my_branch

# 2. Install dependencies
yarn install

# 3. Build packages
yarn build

# 4. Start development mode (optional)
yarn dev  # Watches for changes and rebuilds
```

### Contribution Workflow

#### Before Submitting a Pull Request

1. **Add Tests**: If you've added code that should be tested, add tests
2. **Update Documentation**: If you've changed APIs, update the documentation
3. **Lint Code**: Ensure linting passes via `yarn lint`
4. **Run Tests**: Ensure test suite passes via `yarn test`
5. **Follow Conventions**: Use Prettier formatting (`.prettierrc` in project)
6. **Commit Messages**: Follow [conventional commits](https://www.conventionalcommits.org/) specification

#### Example Commit Messages
```bash
feat: add timeout option for loadable components
fix: resolve chunk loading race condition
docs: update API documentation for ChunkExtractor
test: add tests for error boundary integration
```

### Writing Tests for Loadable Components

#### Test File Structure
```javascript
/* eslint-disable import/no-extraneous-dependencies */
import 'regenerator-runtime/runtime'
import '@testing-library/jest-dom/extend-expect'
import React from 'react'
import { render, cleanup, wait } from '@testing-library/react'
import loadable from './index'

afterEach(cleanup)

describe('#loadable', () => {
  // Test cases here
})
```

#### Component Testing Patterns

**1. Testing Loading States**
```javascript
it('renders nothing without a fallback', () => {
  const Component = loadable(() => new Promise(() => {}))
  const { container } = render(<Component />)
  expect(container).toBeEmpty()
})

it('uses fallback during loading', () => {
  const Component = loadable(
    () => new Promise(() => {}),
    { fallback: 'Loading...' }
  )
  const { container } = render(<Component />)
  expect(container).toHaveTextContent('Loading...')
})
```

**2. Testing Successful Loading**
```javascript
it('mounts component when loaded', async () => {
  const MockComponent = () => 'Loaded Component'
  const load = jest.fn().mockResolvedValue({ default: MockComponent })
  const Component = loadable(load)
  
  const { container } = render(<Component />)
  expect(container).toBeEmpty()
  
  await wait(() => expect(container).toHaveTextContent('Loaded Component'))
  expect(load).toHaveBeenCalledTimes(1)
})
```

**3. Testing Error Handling**
```javascript
it('throws when an error occurs', async () => {
  const load = jest.fn().mockRejectedValue(new Error('Load failed'))
  const Component = loadable(load)
  
  const { container } = render(
    <ErrorBoundary fallback="Error occurred">
      <Component />
    </ErrorBoundary>
  )
  
  await wait(() => expect(container).toHaveTextContent('Error occurred'))
})
```

**4. Testing Cache Behavior**
```javascript
it('caches loaded components', async () => {
  const MockComponent = ({ value }) => `Component: ${value}`
  const load = jest.fn().mockResolvedValue({ default: MockComponent })
  const Component = loadable(load, {
    cacheKey: ({ value }) => value
  })
  
  // First render
  const { container } = render(<Component value="test" />)
  await wait(() => expect(container).toHaveTextContent('Component: test'))
  expect(load).toHaveBeenCalledTimes(1)
  
  // Second render with same cache key
  render(<Component value="test" />, { container })
  expect(load).toHaveBeenCalledTimes(1) // Should not call load again
})
```

#### Server Testing Patterns

**1. Testing ChunkExtractor**
```javascript
describe('ChunkExtractor', () => {
  let extractor
  
  beforeEach(() => {
    extractor = new ChunkExtractor({
      stats: mockStats,
      outputPath: '/dist'
    })
  })
  
  it('should generate script tags', () => {
    extractor.addChunk('my-chunk')
    const scriptTags = extractor.getScriptTags()
    expect(scriptTags).toContain('data-chunk="my-chunk"')
  })
})
```

**2. Testing with Fixtures**
```javascript
import stats from '../__fixtures__/stats.json'

it('should work with real webpack stats', () => {
  const extractor = new ChunkExtractor({ stats })
  // Test with real webpack output
})
```

#### Integration Testing

**1. Full SSR Testing**
```javascript
import { renderToString } from 'react-dom/server'
import { ChunkExtractor } from '@loadable/server'

it('should render server-side with chunks', () => {
  const extractor = new ChunkExtractor({ stats })
  const App = () => {
    const Component = loadable(() => import('./TestComponent'))
    return <Component />
  }
  
  const html = renderToString(extractor.collectChunks(<App />))
  const scriptTags = extractor.getScriptTags()
  
  expect(html).toContain('<!-- SSR content -->')
  expect(scriptTags).toContain('TestComponent')
})
```

### Testing Best Practices

#### 1. Mock External Dependencies
```javascript
// Mock dynamic imports
jest.mock('./MyComponent', () => ({
  default: () => 'Mocked Component'
}))
```

#### 2. Use Snapshot Testing for Complex Output
```javascript
it('should generate correct script elements', () => {
  expect(extractor.getScriptElements()).toMatchInlineSnapshot()
})
```

#### 3. Test Edge Cases
- Network failures
- Malformed webpack stats
- Missing chunks
- Invalid component exports
- Race conditions

#### 4. Performance Testing
```javascript
it('should not cause memory leaks', async () => {
  // Test component cleanup
  const { unmount } = render(<Component />)
  await wait(() => expect(container).toHaveTextContent('loaded'))
  unmount()
  // Verify cleanup
})
```

### Code Quality Standards

#### ESLint Configuration
- Extends Airbnb configuration
- React-specific rules
- Import/export validation
- JSX accessibility rules

#### Prettier Configuration
- Consistent code formatting
- Automatic formatting on commit
- Integration with ESLint

#### Type Safety
- PropTypes validation in development
- TypeScript definitions included
- Runtime type checking where appropriate

This architecture provides a robust, production-ready solution for React code splitting with comprehensive SSR support, making it the recommended choice for complex applications requiring both performance and SEO optimization.
