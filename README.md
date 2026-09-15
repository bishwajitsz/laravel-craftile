# Laravel Craftile

> **Laravel integration for the Craftile visual editor** - Build dynamic, block-based layouts with seamless editor integration.


This package provides Laravel integration for the Craftile Visual Editor, enabling you to create block-based content management systems with Blade templates.

It is a maintained fork of `craftile/laravel`, published by
TradifyLabs. The PHP namespace, service provider, facades, config keys, and publish tags are
unchanged from upstream, so it is a drop-in replacement.

## Relationship to upstream

Only `composer.json` differs from upstream `craftile/laravel` v0.9.2 — the `src/`, `config/`, and
`resources/` trees are identical. Two constraints are relaxed so the package can be used on newer
Laravel releases and with a floating `craftile/core`:

| Requirement | Upstream v0.9.2 | This package |
| --- | --- | --- |
| `illuminate/support` | `^11.0\|^12.0` | `^13.0` |
| `craftile/core` | `self.version` | `^0.9` |

Upstream pins `craftile/core` to `self.version`, which forces core and laravel to be released at
exactly matching version numbers and prevents depending on core `^0.9` with a newer laravel
integration. This fork decouples the two.

Because the namespace is unchanged, do not install this package alongside `craftile/laravel` —
they both provide `Craftile\Laravel\*` and would collide.

## 🚀 Installation

Install via Composer:

```bash
composer require tradifylabs/laravel-craftile
```

If `craftile/laravel` is already required, remove it in the same step so both packages are never
installed at once:

```bash
composer remove craftile/laravel
composer require tradifylabs/laravel-craftile
```

## ⚙️ Configuration

Publish the configuration file:

```bash
php artisan vendor:publish --provider="Craftile\Laravel\CraftileServiceProvider"
```

This will create a `config/craftile.php` file where you can customize:

- Blade directive names
- Component namespace
- Block compiler settings
- Cache configuration

## 📦 Creating Blocks

### 1. Block Class

Create a block by implementing the `BlockInterface`:

```php
<?php

namespace App\Blocks;

use Craftile\Core\Contracts\BlockInterface;
use Craftile\Core\Data\Property\Text;
use Craftile\Core\Data\Property\Boolean;

class Hero implements BlockInterface
{
    public static function getType(): string
    {
        return 'hero';
    }

    public static function getLabel(): string
    {
        return 'Hero Section';
    }

    public static function properties(): array
    {
        return [
            Text::make('title', 'Title')->default('Welcome'),
            Text::make('subtitle', 'Subtitle'),
            Boolean::make('showButton', 'Show Button')->default(true),
        ];
    }

    protected static array $accepts = ['button', 'text']; // Optional: specify accepted child types

    public function render(): mixed
    {
        return view('blocks.hero');
    }
}
```

### 2. Block Template

Create the corresponding Blade template:

```blade
<!-- resources/views/blocks/hero.blade.php -->
<section class="hero bg-gray-900 text-white py-16">
    <div class="container mx-auto px-4">
        <h1 class="text-4xl font-bold mb-4">{{ $block->properties->title }}</h1>

        @if($block->properties->subtitle)
            <p class="text-xl mb-8">{{ $block->properties->subtitle }}</p>
        @endif

        @if($block->properties->showButton)
            @children
        @endif
    </div>
</section>
```

### 3. Register the Block

Register your block in a service provider:

```php
// In AppServiceProvider or dedicated BlockServiceProvider
use Craftile\Laravel\Facades\Craftile;

public function boot()
{
    Craftile::registerBlock(Hero::class);
}
```

## 🎯 Using Blocks in Templates

### Blade Directives

Use blocks directly in your Blade templates:

```blade
{{-- Basic block usage --}}
@craftileBlock('hero', 'hero-1', ['title' => 'Hello World'])

```

### Component Tags

Or use the component syntax:

```blade
<craftile:block type="hero" id="hero-1" :properties="['title' => 'Hello World']" />
```

### JSON/YAML Templates

Create template files that can be rendered directly:

```json
// resources/views/pages/homepage.json
{
    "blocks": {
        "hero-1": {
            "id": "hero-1",
            "type": "hero",
            "properties": {
                "title": "Welcome to our site",
                "subtitle": "Build amazing things with Craftile"
            },
            "children": ["btn-1"]
        },
        "btn-1": {
            "id": "btn-1",
            "type": "button",
            "properties": {
                "text": "Get Started",
                "variant": "primary"
            }
        }
    },
    "regions": [
        {
            "name": "main",
            "blocks": ["hero-1"]
        }
    ]
}
```

Then render in your route or controller:

```php
Route::get('/', function () {
    return view('pages.homepage'); // Will automatically compile the JSON template
});
```

## 🔧 Property Types

Craftile provides several property types for your blocks:

```php
use Craftile\Core\Data\Property\{Text, Textarea, Boolean, Select};

public static function properties(): array
{
    return [
        Text::make('title', 'Title')
            ->default('Default title')
            ->required(),

        Textarea::make('content', 'Content')
            ->default('Enter your content here...'),

        Boolean::make('isVisible', 'Visible')
            ->default(true),

        Select::make('size', 'Size')
            ->options([
                ['value' => 'sm', 'label' => 'Small'],
                ['value' => 'md', 'label' => 'Medium'],
                ['value' => 'lg', 'label' => 'Large'],
            ])
            ->default('md'),
    ];
}
```

## 🎨 Working with Children

Blocks can accept and render child blocks:

```php
// In your block class
protected static array $accepts = ['*']; // Accept any child type
// or
protected static array $accepts = ['button', 'text']; // Accept specific types

// In your block template
@children {{-- Renders all child blocks --}}
```

## 🔧 Advanced Configuration

### Custom Directive Names

Customize directive names in `config/craftile.php`:

```php
'directives' => [
    'craftileBlock' => 'block',        // @block instead of @craftileBlock
    'craftileContent' => 'content',    // @content instead of @craftileContent
],
```

### Component Namespace

Change the component tag namespace:

```php
'components' => [
    'namespace' => 'builder', // <builder:block> instead of <craftile:block>
],
```