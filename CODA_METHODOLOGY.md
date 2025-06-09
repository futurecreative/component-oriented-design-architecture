# CODA: Component-Oriented Design Architecture

> A unified methodology that integrates design intent with component architecture, creating self-contained modules where implementation, interface, styling, and content constraints coexist to ensure visual consistency and code maintainability.

## Core Principles

1. **Unified Encapsulation**: Every component encapsulates interface, implementation, styling, and design intent in a single module
2. **Intent Preservation**: Design decisions and content constraints are explicitly documented and enforced
3. **Clear Contracts**: Component interfaces define both technical contracts and design constraints
4. **Co-location Coherence**: Related code lives together, eliminating scattered implementation and styling
5. **Constraint Enforcement**: Runtime validation preserves design intent across content variations
6. **Extension without Modification**: Components are extended through decoration rather than modification
7. **Single Source of Truth**: For both design tokens and component implementation

## CODA Module Structure

Every CODA module follows this consistent structure:

```tsx
// 1. INTERFACE DEFINITION
export interface IFeatureCard {
  // Technical contract
  render(): JSX.Element;
  updateContent?(props: Partial<FeatureContent>): void;
  
  // Design contract (TypeScript type enforcement)
  readonly constraints: {
    title: ContentConstraint;
    description: ContentConstraint;
    iconName: string[];
  };
}

// Type definitions for content with constraints
type ContentConstraint = {
  minLength?: number;
  maxLength: number;
  format?: string;
  validationRegex?: RegExp;
};

type FeatureContent = {
  title: string;
  description: string;
  iconName: string;
};

// 2. DESIGN INTENT DOCUMENTATION
const DESIGN_INTENT = {
  version: "1.0",
  pattern: "Content-Injected", // CODA pattern type
  purpose: "Present product features with consistent visual hierarchy",
  constraints: {
    title: {
      maxLength: 30,
      format: "Title Case without punctuation",
      examples: ["Seamless Integration", "Advanced Analytics"]
    },
    description: {
      minLength: 20,
      maxLength: 120,
      format: "Complete sentences with proper punctuation",
      examples: ["Connect all your tools with our enterprise integration platform."]
    },
    iconName: {
      options: ["chart", "lock", "globe", "users"],
      rationale: "Limited to ensure visual consistency"
    }
  },
  visualRules: {
    spacing: "Consistent 16px padding between elements",
    alignment: "Icon aligned with title baseline"
  }
};

// 3. DEFAULT CONTENT (varies based on pattern type)
const DEFAULT_CONTENT: FeatureContent = {
  title: "Advanced Analytics",
  description: "Gain deeper insights from your data with our powerful analytics tools.",
  iconName: "chart"
};

// 4. PRIVATE IMPLEMENTATION
class FeatureCardImpl implements IFeatureCard {
  private content: FeatureContent;
  
  constructor(initialContent?: Partial<FeatureContent>) {
    this.content = { ...DEFAULT_CONTENT, ...initialContent };
    this.validateContent();
  }
  
  // Interface implementation
  public get constraints() {
    return {
      title: { maxLength: 30 },
      description: { minLength: 20, maxLength: 120 },
      iconName: ["chart", "lock", "globe", "users"]
    };
  }
  
  public updateContent(newContent: Partial<FeatureContent>): void {
    this.content = { ...this.content, ...newContent };
    this.validateContent();
  }

  // CODA validation
  private validateContent(): void {
    if (process.env.NODE_ENV === 'development') {
      const issues: string[] = [];
      
      if (this.content.title.length > this.constraints.title.maxLength) {
        issues.push(`Title exceeds maximum length (${this.content.title.length}/${this.constraints.title.maxLength})`);
      }
      
      if (this.content.description.length < this.constraints.description.minLength) {
        issues.push(`Description below minimum length (${this.content.description.length}/${this.constraints.description.minLength})`);
      }
      
      if (this.content.description.length > this.constraints.description.maxLength) {
        issues.push(`Description exceeds maximum length (${this.content.description.length}/${this.constraints.description.maxLength})`);
      }
      
      if (!this.constraints.iconName.includes(this.content.iconName)) {
        issues.push(`Invalid icon name "${this.content.iconName}". Must be one of: ${this.constraints.iconName.join(', ')}`);
      }
      
      if (issues.length > 0) {
        console.warn('CODA Design Intent Violations:', issues);
      }
    }
  }
  
  // 5. CO-LOCATED STYLING
  private styles = {
    container: "bg-white rounded-lg p-6 shadow-md hover:shadow-lg transition-shadow",
    iconWrapper: "bg-primary-50 p-4 rounded-full inline-flex mb-4",
    icon: "text-primary-500 w-6 h-6",
    title: "text-xl font-semibold mb-2 text-gray-900",
    description: "text-gray-600"
  };
  
  // Render implementation
  public render(): JSX.Element {
    const { title, description, iconName } = this.content;
    
    return (
      <div className={this.styles.container}>
        <div className={this.styles.iconWrapper}>
          <Icon name={iconName} className={this.styles.icon} />
        </div>
        <h3 className={this.styles.title}>{title}</h3>
        <p className={this.styles.description}>{description}</p>
      </div>
    );
  }
}

// 6. INSTANCE REGISTRY
const registry = new WeakMap<object, IFeatureCard>();

export function getFeatureCard(key: object = {}, initialContent?: Partial<FeatureContent>): IFeatureCard {
  if (!registry.has(key)) {
    registry.set(key, new FeatureCardImpl(initialContent));
  }
  return registry.get(key)!;
}

// 7. REACT COMPONENT WRAPPER
export const FeatureCard: React.FC<{
  title?: string;
  description?: string;
  iconName?: string;
}> = (props) => {
  const featureCard = getFeatureCard({}, props);
  return featureCard.render();
};

// 8. METADATA EXPORTS (for documentation tools)
export const __CODA_METADATA = {
  designIntent: DESIGN_INTENT,
  defaultContent: DEFAULT_CONTENT
};
```

## The Three CODA Patterns

CODA preserves the three fundamental patterns from DICA but implements them within the module structure:

### 1. Component-Owned Pattern

For visually distinctive components where content and design are tightly coupled:

```tsx
// Core difference: No public updateContent method, no props in React wrapper
class HeroSectionImpl implements IHeroSection {
  // Fixed content owned by component
  private readonly content = {
    headline: "Transform Your Digital Experience",
    subheading: "Award-winning solutions for modern businesses",
    ctaText: "Get Started"
  };
  
  // Implementation...
}

// React wrapper takes no content props
export const HeroSection: React.FC = () => {
  const heroSection = getHeroSection();
  return heroSection.render();
};
```

### 2. Content-Injected Pattern

For components that accept content input but enforce design constraints:

```tsx
// Core difference: Has updateContent method, props in React wrapper
class FeatureCardImpl implements IFeatureCard {
  private content: FeatureContent;
  
  constructor(initialContent?: Partial<FeatureContent>) {
    this.content = { ...DEFAULT_CONTENT, ...initialContent };
  }
  
  public updateContent(newContent: Partial<FeatureContent>): void {
    this.content = { ...this.content, ...newContent };
    this.validateContent();
  }
  
  // Implementation...
}

// React wrapper accepts content props
export const FeatureCard: React.FC<{
  title?: string;
  description?: string;
  iconName?: string;
}> = (props) => {
  const featureCard = getFeatureCard({}, props);
  return featureCard.render();
};
```

### 3. Data-Driven Pattern

For components that render collections or datasets with design-aware processing:

```tsx
// Core difference: Works with data collections, has processing functions
class TeamGridImpl implements ITeamGrid {
  private members: TeamMember[] = [];
  
  constructor(initialMembers: TeamMember[] = []) {
    this.setMembers(initialMembers);
  }
  
  public setMembers(members: TeamMember[]): void {
    // Process data through design lens
    this.members = members.map(this.processForDesignConstraints);
    this.validateDataset();
  }
  
  private processForDesignConstraints(member: TeamMember): ProcessedTeamMember {
    return {
      ...member,
      displayName: member.name.length > 20 ? `${member.name.substring(0, 17)}...` : member.name,
      displayRole: member.role.length > 25 ? `${member.role.substring(0, 22)}...` : member.role,
      // Other processing...
    };
  }
  
  // Implementation...
}

// React wrapper accepts data collection
export const TeamGrid: React.FC<{ members: TeamMember[] }> = ({ members }) => {
  const teamGrid = getTeamGrid({}, members);
  return teamGrid.render();
};
```

## Decorator Pattern for Extensions

CODA uses decorators to extend functionality without modifying base components:

```tsx
// BorderedFeatureCard.tsx - A decorator for FeatureCard
export interface IBorderedFeatureCard extends IFeatureCard {
  setBorderColor(color: string): void;
}

class BorderedFeatureCardImpl implements IBorderedFeatureCard {
  private baseCard: IFeatureCard;
  private borderColor: string = 'blue-500';
  
  constructor(baseCard: IFeatureCard) {
    this.baseCard = baseCard;
  }
  
  // Pass-through of base interface
  public get constraints() {
    return this.baseCard.constraints;
  }
  
  public updateContent(content: Partial<FeatureContent>): void {
    this.baseCard.updateContent?.(content);
  }
  
  // Extended functionality
  public setBorderColor(color: string): void {
    this.borderColor = color;
  }
  
  // Override render to add border
  public render(): JSX.Element {
    const baseElement = this.baseCard.render();
    
    // Clone and enhance with border
    return React.cloneElement(baseElement, {
      className: `${baseElement.props.className} border-2 border-${this.borderColor}`
    });
  }
}

// Decorator factory
export function getBorderedFeatureCard(
  key: object = {},
  baseCard?: IFeatureCard,
  initialContent?: Partial<FeatureContent>
): IBorderedFeatureCard {
  const base = baseCard || getFeatureCard({}, initialContent);
  return new BorderedFeatureCardImpl(base);
}

// React wrapper for decorator
export const BorderedFeatureCard: React.FC<{
  title?: string;
  description?: string;
  iconName?: string;
  borderColor?: string;
}> = ({ borderColor = 'blue-500', ...contentProps }) => {
  const baseCard = getFeatureCard({}, contentProps);
  const borderedCard = getBorderedFeatureCard({}, baseCard);
  
  if (borderColor) {
    borderedCard.setBorderColor(borderColor);
  }
  
  return borderedCard.render();
};
```

## Theme Integration

CODA connects to design systems through a Theme Service module:

```tsx
// theme.ts - Theme Service Module
export interface IThemeService {
  getToken(tokenPath: string): string;
  getColorToken(name: string, shade?: number): string;
  getSpacingToken(size: string): string;
  // Other token accessors...
}

class ThemeServiceImpl implements IThemeService {
  private tokens = {
    colors: {
      primary: {
        50: '#f0f9ff',
        500: '#0ea5e9',
        600: '#0284c7'
      },
      gray: {
        600: '#4b5563',
        900: '#111827'
      }
    },
    spacing: {
      xs: '0.25rem',
      sm: '0.5rem',
      md: '1rem',
      lg: '1.5rem',
      xl: '2rem'
    },
    // Other tokens...
  };
  
  public getToken(tokenPath: string): string {
    // Parse path like "colors.primary.500"
    const parts = tokenPath.split('.');
    let value: any = this.tokens;
    
    for (const part of parts) {
      value = value[part];
      if (value === undefined) {
        console.warn(`Theme token not found: ${tokenPath}`);
        return '';
      }
    }
    
    return value;
  }
  
  public getColorToken(name: string, shade = 500): string {
    return this.getToken(`colors.${name}.${shade}`);
  }
  
  public getSpacingToken(size: string): string {
    return this.getToken(`spacing.${size}`);
  }
}

// Service registry
const themeRegistry = new WeakMap<object, IThemeService>();

export function getThemeService(key: object = {}): IThemeService {
  if (!themeRegistry.has(key)) {
    themeRegistry.set(key, new ThemeServiceImpl());
  }
  return themeRegistry.get(key)!;
}
```

## Implementation in Components

Components connect to the theme service:

```tsx
class FeatureCardImpl implements IFeatureCard {
  private themeService: IThemeService;
  
  constructor(initialContent?: Partial<FeatureContent>) {
    this.themeService = getThemeService();
    // Rest of constructor...
  }
  
  private styles = {
    container: `bg-white rounded-lg p-${this.themeService.getSpacingToken('md')} shadow-md`,
    title: `text-xl font-semibold mb-${this.themeService.getSpacingToken('sm')} text-${this.themeService.getColorToken('gray', 900)}`,
    // Other styles...
  };
  
  // Implementation...
}
```

## Testing CODA Components

CODA's module pattern makes testing straightforward:

```tsx
// FeatureCard.test.tsx
describe('FeatureCard', () => {
  // 1. Interface Contract Tests
  it('implements IFeatureCard interface correctly', () => {
    const card = getFeatureCard();
    expect(card.render).toBeDefined();
    expect(card.updateContent).toBeDefined();
    expect(card.constraints).toBeDefined();
  });
  
  // 2. Design Intent Tests
  it('validates content against design constraints', () => {
    const consoleWarnSpy = jest.spyOn(console, 'warn').mockImplementation();
    
    const card = getFeatureCard();
    card.updateContent({
      title: 'This title is way too long and exceeds the maximum allowed length for the design',
      description: 'Too short'
    });
    
    expect(consoleWarnSpy).toHaveBeenCalledWith(
      'CODA Design Intent Violations:',
      expect.arrayContaining([
        expect.stringContaining('Title exceeds maximum length'),
        expect.stringContaining('Description below minimum length')
      ])
    );
  });
  
  // 3. Visual Regression Tests
  it('renders consistently', () => {
    const { container } = render(<FeatureCard />);
    expect(container).toMatchSnapshot();
  });
});
```

## Migration Path to CODA

For teams adopting CODA, follow this incremental approach:

1. **Audit Phase**: Categorize existing components into the three CODA patterns
2. **Scaffolding**: Create CODA interfaces for each component category
3. **Module Conversion**: One by one, refactor components into CODA modules
4. **Design Intent Documentation**: Add constraints and validation
5. **Theme Service Implementation**: Connect to design system
6. **Registry Implementation**: Add service registries and factories
7. **Testing**: Add contract and design intent tests

## Benefits of CODA

1. **Unified Documentation**: Design and technical documentation live together
2. **Simplified Onboarding**: New developers understand both code and design intent
3. **Reduced Bugs**: Runtime validation prevents design regressions
4. **Easier Maintenance**: Changes happen in one place, not scattered across files
5. **Better Testing**: Contract-based testing for both implementation and design
6. **Clearer Design System Connection**: Explicit theme service integration

CODA integrates the best aspects of component architecture and design intent preservation into a cohesive methodology that's practical, repeatable, and powerful for modern UI development. 