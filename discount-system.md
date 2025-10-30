# Discount System
- This piece of snippet implements a flexible discount system for an e-commerce platform
- It can handle multiple discount types and combine them intelligently.

```
interface CartItem {
  id: string;
  name: string;
  price: number;
  category: string;
  quantity: number;
}

interface Discount {
  id: string;
  name: string;
  type: 'percentage' | 'fixed' | 'bogof' | 'category';
  value: number;
  applicableCategories?: string[];
  minCartValue?: number;
  maxDiscount?: number;
}

class DiscountEngine {
  We apply implementation here
}

```

## AI/ML Personalised Product Implementation Architecture
https://github.com/kukuu/AI-ML-recommendation-personalised-product/blob/main/README.md

## Handling Regional Pricing

https://github.com/kukuu/AI-ML-recommendation-personalised-product/blob/main/handling-regional-pricing-with-configs.md
