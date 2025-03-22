<think>
I need to create a modern e-commerce website based on balancecleanse.com, using Next.js with TypeScript, Tailwind CSS, and Next-UI. The focus is on creating a user-friendly interface with dynamic visual feedback for mouse interactions.

The prompt already provides a detailed guide including project setup, structure, and code snippets for key components. Let me analyze this carefully and develop a complete solution.

First, let me understand the key requirements:
- Next.js with TypeScript for the framework
- Tailwind CSS for styling
- Next-UI for component library
- Dynamic visual feedback on hover
- Modern e-commerce functionality

I'll need to create:
1. Project setup and configuration
2. Core layout components (Header, Footer)
3. Product listing and detail pages
4. Cart and checkout functionality
5. Supporting pages (About, Contact)
6. State management for cart

Let me follow the provided guide and expand where needed to create a complete solution. I'll make sure to implement engaging hover effects and transitions throughout the UI.
</think>

# Modern E-commerce Website Using Next.js, TypeScript, Tailwind CSS, and NextUI

Below you'll find a complete implementation for a modern e-commerce website inspired by balancecleanse.com. This implementation leverages Next.js with TypeScript, Tailwind CSS for styling, and NextUI for UI components, with a focus on creating dynamic visual feedback for an engaging user experience.

## Project Setup and Initialization

First, let's create a new Next.js project and install the necessary dependencies:

```bash
npx create-next-app@latest balancecleanse-clone --typescript
cd balancecleanse-clone
npm install @nextui-org/react tailwindcss postcss autoprefixer
```

## Configuration Files

### Tailwind Configuration (`tailwind.config.js`)

```javascript
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: [
    "./pages/**/*.{js,ts,jsx,tsx,mdx}",
    "./components/**/*.{js,ts,jsx,tsx,mdx}",
    "./app/**/*.{js,ts,jsx,tsx,mdx}",
    "./node_modules/@nextui-org/theme/dist/**/*.{js,ts,jsx,tsx}", // Important for NextUI
  ],
  theme: {
    extend: {
      // Custom colors based on balancecleanse.com
      colors: {
        'primary': '#4A8072', // Soft green
        'secondary': '#E8DED1', // Beige
        'accent': '#D97D54', // Orange accent
        'dark': '#2C3E50', // Dark blue-gray
      },
      fontFamily: {
        'sans': ['Inter', 'sans-serif'],
      },
      // Custom transitions for hover effects
      transitionProperty: {
        'height': 'height',
        'spacing': 'margin, padding',
      }
    },
  },
  darkMode: "class", // Important for NextUI dark mode
  plugins: [],
};
```

### PostCSS Configuration (`postcss.config.js`)

```javascript
module.exports = {
  plugins: {
    tailwindcss: {},
    autoprefixer: {},
  },
};
```

### NextUI Configuration (`app/layout.tsx`)

```typescript
import type { Metadata } from 'next'
import { Inter } from 'next/font/google'
import './globals.css'
import { NextUIProvider } from '@nextui-org/react'
import Header from '@/components/layout/Header'
import Footer from '@/components/layout/Footer'
import { CartProvider } from '@/lib/context/CartContext'

const inter = Inter({ subsets: ['latin'] })

export const metadata: Metadata = {
  title: 'Balance Cleanse - Natural Wellness Products',
  description: 'Premium organic products to support your wellness journey and cleanse your body naturally.',
}

export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="en">
      <body className={inter.className}>
        <CartProvider>
          <NextUIProvider>
            <div className="flex flex-col min-h-screen">
              <Header />
              <main className="flex-grow">
                {children}
              </main>
              <Footer />
            </div>
          </NextUIProvider>
        </CartProvider>
      </body>
    </html>
  )
}
```

### Global Styles (`app/globals.css`)

```css
@tailwind base;
@tailwind components;
@tailwind utilities;

:root {
  --foreground-rgb: 0, 0, 0;
  --background-rgb: 255, 255, 255;
}

body {
  color: rgb(var(--foreground-rgb));
  background: rgb(var(--background-rgb));
}

/* Custom animations */
@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}

@keyframes slideUp {
  from { transform: translateY(20px); opacity: 0; }
  to { transform: translateY(0); opacity: 1; }
}

.animate-fade-in {
  animation: fadeIn 0.5s ease-in-out;
}

.animate-slide-up {
  animation: slideUp 0.5s ease-out;
}

/* Custom utility classes */
.hover-scale {
  @apply transition-transform duration-300 hover:scale-105;
}

.hover-lift {
  @apply transition-all duration-300 hover:-translate-y-1 hover:shadow-lg;
}
```

## Type Definitions

Let's define our core types in a dedicated file:

### `lib/types.ts`

```typescript
// Product-related types
export interface Product {
  id: string;
  slug: string;
  name: string;
  description: string;
  price: number;
  images: string[];
  category: string;
  inStock: boolean;
  featured?: boolean;
  attributes?: {
    [key: string]: string;
  };
}

// Cart-related types
export interface CartItem {
  productId: string;
  quantity: number;
  product: Product;
}

export interface Cart {
  items: CartItem[];
  subtotal: number;
  tax: number;
  shipping: number;
  total: number;
}

// User-related types (for future auth implementation)
export interface User {
  id: string;
  name: string;
  email: string;
}
```

## Context for State Management

### `lib/context/CartContext.tsx`

```typescript
import { createContext, useContext, useState, useEffect, ReactNode } from 'react';
import { Cart, CartItem, Product } from '../types';

interface CartContextType {
  cart: Cart;
  addToCart: (product: Product, quantity: number) => void;
  removeFromCart: (productId: string) => void;
  updateQuantity: (productId: string, quantity: number) => void;
  clearCart: () => void;
  isInCart: (productId: string) => boolean;
}

const CartContext = createContext<CartContextType | undefined>(undefined);

const initCart = (): Cart => ({
  items: [],
  subtotal: 0,
  tax: 0,
  shipping: 0,
  total: 0,
});

export const CartProvider = ({ children }: { children: ReactNode }) => {
  const [cart, setCart] = useState<Cart>(initCart());

  // Load cart from localStorage on mount
  useEffect(() => {
    if (typeof window !== 'undefined') {
      const savedCart = localStorage.getItem('cart');
      if (savedCart) {
        try {
          setCart(JSON.parse(savedCart));
        } catch (e) {
          console.error("Failed to parse cart from localStorage", e);
        }
      }
    }
  }, []);

  // Save cart to localStorage when it changes
  useEffect(() => {
    if (typeof window !== 'undefined') {
      localStorage.setItem('cart', JSON.stringify(cart));
    }
  }, [cart]);

  // Calculate totals whenever cart items change
  useEffect(() => {
    const subtotal = cart.items.reduce(
      (sum, item) => sum + item.product.price * item.quantity, 
      0
    );
    const tax = subtotal * 0.08; // Example tax rate
    const shipping = subtotal > 100 ? 0 : 10; // Free shipping over $100
    const total = subtotal + tax + shipping;
    
    setCart(prev => ({
      ...prev,
      subtotal,
      tax,
      shipping,
      total
    }));
  }, [cart.items]);

  const addToCart = (product: Product, quantity: number) => {
    setCart(prev => {
      const existingItemIndex = prev.items.findIndex(
        item => item.productId === product.id
      );

      let newItems;
      
      if (existingItemIndex >= 0) {
        // Update existing item
        newItems = [...prev.items];
        newItems[existingItemIndex] = {
          ...newItems[existingItemIndex],
          quantity: newItems[existingItemIndex].quantity + quantity
        };
      } else {
        // Add new item
        newItems = [
          ...prev.items,
          { productId: product.id, quantity, product }
        ];
      }

      return {
        ...prev,
        items: newItems
      };
    });
  };

  const removeFromCart = (productId: string) => {
    setCart(prev => ({
      ...prev,
      items: prev.items.filter(item => item.productId !== productId)
    }));
  };

  const updateQuantity = (productId: string, quantity: number) => {
    if (quantity <= 0) {
      removeFromCart(productId);
      return;
    }

    setCart(prev => ({
      ...prev,
      items: prev.items.map(item => 
        item.productId === productId 
          ? { ...item, quantity } 
          : item
      )
    }));
  };

  const clearCart = () => {
    setCart(initCart());
  };

  const isInCart = (productId: string) => {
    return cart.items.some(item => item.productId === productId);
  };

  return (
    <CartContext.Provider 
      value={{ 
        cart, 
        addToCart, 
        removeFromCart, 
        updateQuantity, 
        clearCart,
        isInCart
      }}
    >
      {children}
    </CartContext.Provider>
  );
};

export const useCart = () => {
  const context = useContext(CartContext);
  if (context === undefined) {
    throw new Error('useCart must be used within a CartProvider');
  }
  return context;
};
```

## Custom Hooks

### `lib/hooks/useProducts.ts`

```typescript
import { useState, useEffect } from 'react';
import { Product } from '../types';

// In a real application, this would fetch from an API
// For demo purposes, we'll use dummy data
const dummyProducts: Product[] = [
  {
    id: '1',
    slug: 'balance-cleanse-starter-kit',
    name: 'Balance Cleanse Starter Kit',
    description: 'Begin your wellness journey with our essential starter kit. Includes our signature Balance Tea and Digestive Support supplements.',
    price: 39.99,
    images: ['/images/product-1.jpg'],
    category: 'Kits',
    inStock: true,
    featured: true,
    attributes: {
      'Weight': '250g',
      'Serves': '30 days'
    }
  },
  {
    id: '2',
    slug: 'organic-detox-tea',
    name: 'Organic Detox Tea',
    description: 'Our premium blend of organic herbs designed to support your body\'s natural detoxification processes.',
    price: 24.99,
    images: ['/images/product-2.jpg'],
    category: 'Teas',
    inStock: true,
    attributes: {
      'Weight': '100g',
      'Serves': '20 cups'
    }
  },
  {
    id: '3',
    slug: 'digestive-enzyme-complex',
    name: 'Digestive Enzyme Complex',
    description: 'Support optimal digestion with our plant-based enzyme complex. Perfect for those experiencing occasional digestive discomfort.',
    price: 29.99,
    images: ['/images/product-3.jpg'],
    category: 'Supplements',
    inStock: true,
    attributes: {
      'Count': '60 capsules',
      'Serving': '2 capsules daily'
    }
  },
  {
    id: '4',
    slug: 'gut-health-probiotic',
    name: 'Gut Health Probiotic',
    description: 'Advanced probiotic formula with 15 strains of beneficial bacteria to support gut health and immune function.',
    price: 34.99,
    images: ['/images/product-4.jpg'],
    category: 'Supplements',
    inStock: true,
    featured: true,
    attributes: {
      'Count': '30 capsules',
      'Serving': '1 capsule daily'
    }
  },
  {
    id: '5',
    slug: 'complete-cleanse-program',
    name: 'Complete Cleanse Program',
    description: 'Our comprehensive 30-day cleanse program includes everything you need for a full-body reset: detox tea, digestive enzymes, probiotics, and a detailed guide.',
    price: 89.99,
    images: ['/images/product-5.jpg'],
    category: 'Programs',
    inStock: true,
    featured: true,
    attributes: {
      'Duration': '30 days',
      'Includes': 'Tea, supplements, guide'
    }
  },
  {
    id: '6',
    slug: 'liver-support-formula',
    name: 'Liver Support Formula',
    description: 'Support your liver\'s vital functions with our blend of milk thistle, dandelion root, and other traditional liver-supporting herbs.',
    price: 27.99,
    images: ['/images/product-6.jpg'],
    category: 'Supplements',
    inStock: true,
    attributes: {
      'Count': '60 capsules',
      'Serving': '2 capsules daily'
    }
  },
];

export function useProducts() {
  const [products, setProducts] = useState<Product[]>([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<Error | null>(null);

  useEffect(() => {
    // Simulate API fetch with a timeout
    const fetchProducts = async () => {
      try {
        // In a real app, this would be an API call
        setTimeout(() => {
          setProducts(dummyProducts);
          setLoading(false);
        }, 500);
      } catch (err) {
        setError(err instanceof Error ? err : new Error('Unknown error'));
        setLoading(false);
      }
    };

    fetchProducts();
  }, []);

  const getProductBySlug = (slug: string) => {
    return products.find(product => product.slug === slug) || null;
  };

  const getFeaturedProducts = () => {
    return products.filter(product => product.featured);
  };

  const getProductsByCategory = (category: string) => {
    return products.filter(product => product.category === category);
  };

  return {
    products,
    loading,
    error,
    getProductBySlug,
    getFeaturedProducts,
    getProductsByCategory
  };
}
```

## UI Components

### `components/ui/CartIcon.tsx`

```typescript
import { useCart } from '@/lib/hooks/useCart';

export default function CartIcon() {
  const { cart } = useCart();
  const itemCount = cart.items.reduce((total, item) => total + item.quantity, 0);

  return (
    <div className="relative transition-transform hover:scale-110">
      <svg 
        xmlns="http://www.w3.org/2000/svg" 
        className="h-6 w-6" 
        fill="none" 
        viewBox="0 0 24 24" 
        stroke="currentColor"
      >
        <path 
          strokeLinecap="round" 
          strokeLinejoin="round" 
          strokeWidth={2} 
          d="M3 3h2l.4 2M7 13h10l4-8H5.4M7 13L5.4 5M7 13l-2.293 2.293c-.63.63-.184 1.707.707 1.707H17m0 0a2 2 0 100 4 2 2 0 000-4zm-8 2a2 2 0 11-4 0 2 2 0 014 0z" 
        />
      </svg>
      {itemCount > 0 && (
        <span className="absolute -top-2 -right-2 bg-primary text-white rounded-full w-5 h-5 flex items-center justify-center text-xs animate-fade-in">
          {itemCount}
        </span>
      )}
    </div>
  );
}
```

### `components/layout/Header.tsx`

```typescript
'use client';

import { useState } from 'react';
import Link from 'next/link';
import { 
  Navbar, 
  NavbarBrand, 
  NavbarContent, 
  NavbarItem, 
  NavbarMenuToggle,
  NavbarMenu,
  NavbarMenuItem,
  Dropdown, 
  DropdownTrigger, 
  DropdownMenu, 
  DropdownItem,
  Button
} from "@nextui-org/react";
import CartIcon from '../ui/CartIcon';
import { useCart } from '@/lib/hooks/useCart';

export default function Header() {
  const [isMenuOpen, setIsMenuOpen] = useState(false);
  const { cart } = useCart();
  const itemCount = cart.items.reduce((total, item) => total + item.quantity, 0);

  // Define categories for dropdown
  const categories = ["Teas", "Supplements", "Kits", "Programs"];

  return (
    <Navbar 
      isBordered 
      isMenuOpen={isMenuOpen} 
      onMenuOpenChange={setIsMenuOpen}
      className="bg-white/70 backdrop-blur-md border-b border-gray-200"
    >
      <NavbarContent className="sm:hidden" justify="start">
        <NavbarMenuToggle aria-label={isMenuOpen ? "Close menu" : "Open menu"} />
      </NavbarContent>

      <NavbarContent className="sm:hidden pr-3" justify="center">
        <NavbarBrand>
          <Link href="/" className="font-bold text-inherit">
            Balance Cleanse
          </Link>
        </NavbarBrand>
      </NavbarContent>

      <NavbarContent className="hidden sm:flex gap-4" justify="start">
        <NavbarBrand>
          <Link href="/" className="font-bold text-xl text-inherit transition-transform hover:scale-105 inline-block">
            Balance Cleanse
          </Link>
        </NavbarBrand>
      </NavbarContent>

      <NavbarContent className="hidden sm:flex gap-4" justify="center">
        <NavbarItem>
          <Link 
            href="/" 
            className="text-base transition-all hover:text-primary hover:font-semibold hover:scale-105 inline-block"
          >
            Home
          </Link>
        </NavbarItem>
        <Dropdown>
          <NavbarItem>
            <DropdownTrigger>
              <Button
                disableRipple
                className="p-0 bg-transparent data-[hover=true]:bg-transparent text-base transition-all hover:text-primary hover:font-semibold hover:scale-105"
                endContent={
                  <svg 
                    xmlns="http://www.w3.org/2000/svg" 
                    className="h-4 w-4 mt-0.5" 
                    fill="none" 
                    viewBox="0 0 24 24" 
                    stroke="currentColor"
                  >
                    <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M19 9l-7 7-7-7" />
                  </svg>
                }
                radius="sm"
                variant="light"
              >
                Products
              </Button>
            </DropdownTrigger>
          </NavbarItem>
          <DropdownMenu aria-label="Product Categories">
            <DropdownItem key="all" as={Link} href="/products">
              All Products
            </DropdownItem>
            {categories.map((category) => (
              <DropdownItem key={category} as={Link} href={`/products?category=${category.toLowerCase()}`}>
                {category}
              </DropdownItem>
            ))}
          </DropdownMenu>
        </Dropdown>
        <NavbarItem>
          <Link 
            href="/about" 
            className="text-base transition-all hover:text-primary hover:font-semibold hover:scale-105 inline-block"
          >
            About
          </Link>
        </NavbarItem>
        <NavbarItem>
          <Link 
            href="/contact" 
            className="text-base transition-all hover:text-primary hover:font-semibold hover:scale-105 inline-block"
          >
            Contact
          </Link>
        </NavbarItem>
      </NavbarContent>

      <NavbarContent justify="end">
        <NavbarItem>
          <Link 
            href="/cart" 
            className="transition-transform hover:scale-110 inline-block"
          >
            <div className="relative">
              <CartIcon />
            </div>
          </Link>
        </NavbarItem>
        <NavbarItem>
          <Button 
            as={Link} 
            color="primary" 
            href="/checkout" 
            variant="flat"
            className="transition-transform hover:scale-105"
            isDisabled={itemCount === 0}
          >
            Checkout
          </Button>
        </NavbarItem>
      </NavbarContent>

      <NavbarMenu>
        <NavbarMenuItem>
          <Link 
            href="/" 
            className="w-full py-2 text-lg"
            onClick={() => setIsMenuOpen(false)}
          >
            Home
          </Link>
        </NavbarMenuItem>
        <NavbarMenuItem>
          <Link 
            href="/products" 
            className="w-full py-2 text-lg"
            onClick={() => setIsMenuOpen(false)}
          >
            All Products
          </Link>
        </NavbarMenuItem>
        {categories.map((category, index) => (
          <NavbarMenuItem key={`${category}-${index}`}>
            <Link 
              href={`/products?category=${category.toLowerCase()}`}
              className="w-full py-2 text-lg"
              onClick={() => setIsMenuOpen(false)}
            >
              {category}
            </Link>
          </NavbarMenuItem>
        ))}
        <NavbarMenuItem>
          <Link 
            href="/about" 
            className="w-full py-2 text-lg"
            onClick={() => setIsMenuOpen(false)}
          >
            About
          </Link>
        </NavbarMenuItem>
        <NavbarMenuItem>
          <Link 
            href="/contact" 
            className="w-full py-2 text-lg"
            onClick={() => setIsMenuOpen(false)}
          >
            Contact
          </Link>
        </NavbarMenuItem>
      </NavbarMenu>
    </Navbar>
  );
}
```

### `components/layout/Footer.tsx`

```typescript
import Link from 'next/link';
import { Input, Button } from "@nextui-org/react";

export default function Footer() {
  return (
    <footer className="bg-gray-50 border-t border-gray-200 pt-12 pb-8">
      <div className="container mx-auto px-4">
        <div className="grid grid-cols-1 md:grid-cols-4 gap-8">
          <div>
            <h3 className="text-lg font-semibold mb-4">Balance Cleanse</h3>
            <p className="text-gray-600 mb-4">
              Your partner in wellness and natural health solutions. We're dedicated to helping you achieve balance and vitality.
            </p>
            <div className="flex space-x-4">
              <a 
                href="https://facebook.com" 
                target="_blank" 
                rel="noopener noreferrer"
                className="text-gray-400 hover:text-primary transition-colors duration-300"
              >
                <svg className="h-6 w-6" fill="currentColor" viewBox="0 0 24 24" aria-hidden="true">
                  <path fillRule="evenodd" d="M22 12c0-5.523-4.477-10-10-10S2 6.477 2 12c0 4.991 3.657 9.128 8.438 9.878v-6.987h-2.54V12h2.54V9.797c0-2.506 1.492-3.89 3.777-3.89 1.094 0 2.238.195 2.238.195v2.46h-1.26c-1.243 0-1.63.771-1.63 1.562V12h2.773l-.443 2.89h-2.33v6.988C18.343 21.128 22 16.991 22 12z" clipRule="evenodd" />
                </svg>
              </a>
              <a 
                href="https://instagram.com" 
                target="_blank" 
                rel="noopener noreferrer"
                className="text-gray-400 hover:text-primary transition-colors duration-300"
              >
                <svg className="h-6 w-6" fill="currentColor" viewBox="0 0 24 24" aria-hidden="true">
                  <path fillRule="evenodd" d="M12.315 2c2.43 0 2.784.013 3.808.06 1.064.049 1.791.218 2.427.465a4.902 4.902 0 011.772 1.153 4.902 4.902 0 011.153 1.772c.247.636.416 1.363.465 2.427.048 1.067.06 1.407.06 4.123v.08c0 2.643-.012 2.987-.06 4.043-.049 1.064-.218 1.791-.465 2.427a4.902 4.902 0 01-1.153 1.772 4.902 4.902 0 01-1.772 1.153c-.636.247-1.363.416-2.427.465-1.067.048-1.407.06-4.123.06h-.08c-2.643 0-2.987-.012-4.043-.06-1.064-.049-1.791-.218-2.427-.465a4.902 4.902 0 01-1.772-1.153 4.902 4.902 0 01-1.153-1.772c-.247-.636-.416-1.363-.465-2.427-.047-1.024-.06-1.379-.06-3.808v-.63c0-2.43.013-2.784.06-3.808.049-1.064.218-1.791.465-2.427a4.902 4.902 0 011.153-1.772A4.902 4.902 0 015.45 2.525c.636-.247 1.363-.416 2.427-.465C8.901 2.013 9.256 2 11.685 2h.63zm-.081 1.802h-.468c-2.456 0-2.784.011-3.807.058-.975.045-1.504.207-1.857.344-.467.182-.8.398-1.15.748-.35.35-.566.683-.748 1.15-.137.353-.3.882-.344 1.857-.047 1.023-.058 1.351-.058 3.807v.468c0 2.456.011 2.784.058 3.807.045.975.207 1.504.344 1.857.182.466.399.8.748 1.15.35.35.683.566 1.15.748.353.137.882.3 1.857.344 1.054.048 1.37.058 4.041.058h.08c2.597 0 2.917-.01 3.96-.058.976-.045 1.505-.207 1.858-.344.466-.182.8-.398 1.15-.748.35-.35.566-.683.748-1.15.137-.353.3-.882.344-1.857.048-1.055.058-1.37.058-4.041v-.08c0-2.597-.01-2.917-.058-3.96-.045-.976-.207-1.505-.344-1.858a3.097 3.097 0 00-.748-1.15 3.098 3.098 0 00-1.15-.748c-.353-.137-.882-.3-1.857-.344-1.023-.047-1.351-.058-3.807-.058zM12 6.865a5.135 5.135 0 110 10.27 5.135 5.135 0 010-10.27zm0 1.802a3.333 3.333 0 100 6.666 3.333 3.333 0 000-6.666zm5.338-3.205a1.2 1.2 0 110 2.4 1.2 1.2 0 010-2.4z" clipRule="evenodd" />
                </svg>
              </a>
              <a 
                href="https://twitter.com" 
                target="_blank" 
                rel="noopener noreferrer"
                className="text-gray-400 hover:text-primary transition-colors duration-300"
              >
                <svg className="h-6 w-6" fill="currentColor" viewBox="0 0 24 24" aria-hidden="true">
                  <path d="M8.29 20.251c7.547 0 11.675-6.253 11.675-11.675 0-.178 0-.355-.012-.53A8.348 8.348 0 0022 5.92a8.19 8.19 0 01-2.357.646 4.118 4.118 0 001.804-2.27 8.224 8.224 0 01-2.605.996 4.107 4.107 0 00-6.993 3.743 11.65 11.65 0 01-8.457-4.287 4.106 4.106 0 001.27 5.477A4.072 4.072 0 012.8 9.713v.052a4.105 4.105 0 003.292 4.022 4.095 4.095 0 01-1.853.07 4.108 4.108 0 003.834 2.85A8.233 8.233 0 012 18.407a11.616 11.616 0 006.29 1.84" />
                </svg>
              </a>
            </div>
          </div>
          
          <div>
            <h3 className="text-lg font-semibold mb-4">Quick Links</h3>
            <ul className="space-y-2">
              <li>
                <Link href="/" className="text-gray-600 hover:text-primary hover:underline transition-colors duration-200">
                  Home
                </Link>
              </li>
              <li>
                <Link href="/products" className="text-gray-600 hover:text-primary hover:underline transition-colors duration-200">
                  Products
                </Link>
              </li>
              <li>
                <Link href="/about" className="text-gray-600 hover:text-primary hover:underline transition-colors duration-200">
                  About Us
                </Link>
              </li>
              <li>
                <Link href="/contact" className="text-gray-600 hover:text-primary hover:underline transition-colors duration-200">
                  Contact
                </Link>
              </li>
            </ul>
          </div>
          
          <div>
            <h3 className="text-lg font-semibold mb-4">Customer Service</h3>
            <ul className="space-y-2">
              <li>
                <Link href="/shipping" className="text-gray-600 hover:text-primary hover:underline transition-colors duration-200">
                  Shipping & Returns
                </Link>
              </li>
              <li>
                <Link href="/faq" className="text-gray-600 hover:text-primary hover:underline transition-colors duration-200">
                  FAQ
                </Link>
              </li>
              <li>
                <Link href="/privacy" className="text-gray-600 hover:text-primary hover:underline transition-colors duration-200">
                  Privacy Policy
                </Link>
              </li>
              <li>
                <Link href="/terms" className="text-gray-600 hover:text-primary hover:underline transition-colors duration-200">
                  Terms of Service
                </Link>
              </li>
            </ul>
          </div>
          
          <div>
            <h3 className="text-lg font-semibold mb-4">Newsletter</h3>
            <p className="text-gray-600 mb-4">
              Subscribe to our newsletter for the latest product updates and wellness tips.
            </p>
            <div className="flex flex-col space-y-3">
              <Input
                type="email"
                placeholder="Your email address"
                className="max-w-xs hover-lift"
              />
              <Button 
                color="primary" 
                className="max-w-[120px] transition-transform hover:scale-105"
              >
                Subscribe
              </Button>
            </div>
          </div>
        </div>
        
        <div className="mt-8 pt-8 border-t border-gray-200">
          <p className="text-center text-gray-500 text-sm">
            © {new Date().getFullYear()} Balance Cleanse. All rights reserved.
          </p>
        </div>
      </div>
    </footer>
  );
}
```

### `components/products/ProductCard.tsx`

```typescript
'use client';

import { useState } from 'react';
import Link from 'next/link';
import { Card, CardHeader, CardBody, CardFooter, Image, Button, Badge } from "@nextui-org/react";
import { Product } from '@/lib/types';
import { useCart } from '@/lib/hooks/useCart';

interface ProductCardProps {
  product: Product;
}

export default function ProductCard({ product }: ProductCardProps) {
  const { addToCart, isInCart } = useCart();
  const [isHovered, setIsHovered] = useState(false);
  
  const handleAddToCart = (e: React.MouseEvent) => {
    e.preventDefault(); // Prevent navigation
    addToCart(product, 1);
  };

  return (
    <Card 
      className="w-[300px] h-[400px] transition-all duration-300"
      style={{ 
        transform: isHovered ? 'scale(1.05)' : 'scale(1)',
        boxShadow: isHovered ? '0 10px 25px rgba(0, 0, 0, 0.1)' : '0 4px 10px rgba(0, 0, 0, 0.05)'
      }}
      onMouseEnter={() => setIsHovered(true)}
      onMouseLeave={() => setIsHovered(false)}
      isPressable
      as={Link}
      href={`/products/${product.slug}`}
    >
      <CardHeader className="absolute z-10 top-1 flex-col items-start">
        <Badge color="primary" variant="flat">
          {product.category}
        </Badge>
        {!product.inStock && (
          <Badge color="danger" variant="flat" className="mt-1">
            Out of Stock
          </Badge>
        )}
      </CardHeader>
      <Image
        removeWrapper
        alt={product.name}
        className="z-0 w-full h-full object-cover transition-transform duration-500"
        style={{ transform: isHovered ? 'scale(1.1)' : 'scale(1)' }}
        src={product.images[0] || '/images/placeholder.jpg'}
        width={300}
        height={
