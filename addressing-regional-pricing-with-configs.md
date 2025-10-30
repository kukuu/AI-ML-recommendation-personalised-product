# Addressing Regional Pricing

```
interface RegionalPricing {
  region: string;
  currency: string;
  priceModifier: number;
  taxRate: number;
  applicableCategories: string[];
}

class RegionalDiscountStrategy implements DiscountStrategy {
  private regionalPricing: Map<string, RegionalPricing>;
  
  constructor(regionalPricing: RegionalPricing[]) {
    this.regionalPricing = new Map(regionalPricing.map(rp => [rp.region, rp]));
  }

  calculate(cart: CartItem[], discount: Discount, region: string): number {
    const regionalConfig = this.regionalPricing.get(region);
    if (!regionalConfig) {
      throw new Error(`Regional pricing not configured for: ${region}`);
    }

    const applicableItems = cart.filter(item => 
      regionalConfig.applicableCategories.includes(item.category)
    );

    const baseDiscount = this.calculateBaseDiscount(applicableItems, discount);
    const regionalAdjusted = baseDiscount * regionalConfig.priceModifier;
    
    return this.applyRegionalLimits(regionalAdjusted, regionalConfig, discount);
  }

  private calculateBaseDiscount(items: CartItem[], discount: Discount): number {
    const total = items.reduce((sum, item) => sum + (item.price * item.quantity), 0);
    
    switch (discount.type) {
      case 'percentage':
        return total * (discount.value / 100);
      case 'fixed':
        return discount.value;
      case 'category':
        return this.calculateCategoryDiscount(items, discount);
      default:
        return 0;
    }
  }

  private calculateCategoryDiscount(items: CartItem[], discount: Discount): number {
    return items.reduce((sum, item) => {
      if (discount.applicableCategories?.includes(item.category)) {
        return sum + (item.price * item.quantity * (discount.value / 100));
      }
      return sum;
    }, 0);
  }

  private applyRegionalLimits(
    discount: number, 
    regionalConfig: RegionalPricing, 
    originalDiscount: Discount
  ): number {
    let finalDiscount = discount;
    
    // Apply regional maximum discount if specified
    if (regionalConfig.priceModifier < 1 && originalDiscount.maxDiscount) {
      finalDiscount = Math.min(finalDiscount, originalDiscount.maxDiscount);
    }
    
    return Math.max(0, finalDiscount);
  }
}

// Usage example:
const regionalPricingConfig: RegionalPricing[] = [
  {
    region: 'US',
    currency: 'USD',
    priceModifier: 1.0,
    taxRate: 0.08,
    applicableCategories: ['electronics', 'clothing']
  },
  {
    region: 'EU',
    currency: 'EUR', 
    priceModifier: 0.9,
    taxRate: 0.2,
    applicableCategories: ['electronics', 'clothing', 'books']
  },
  {
    region: 'UK',
    currency: 'GBP',
    priceModifier: 0.85,
    taxRate: 0.2,
    applicableCategories: ['electronics']
  }
];

// Initialize strategy
const regionalStrategy = new RegionalDiscountStrategy(regionalPricingConfig);

// Calculate discount for specific region
const discount = regionalStrategy.calculate(cartItems, discountConfig, 'EU');

```
