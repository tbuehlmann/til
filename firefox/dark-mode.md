# Dark Mode

Firefox can be setup with a dark theme:

```
Add-Ons --> Themes --> Enabled "Dark"
```

However, this will only change the appearance of Firefox elements like the address bar and tabs. It doesn't tell websites that a dark mode is preferred.

In order to do so, open `about:config` in the browser address bar and add an integer option named `ui.systemUsesDarkTheme` with the value `1`. With that, CSS knows about the preference and can display things appropriately.

## CSS

Using CSS, we can use the browser's dark mode preference like this:

```css
@media (prefers-color-scheme: dark) {
  body {
    background-color: black;
  }
}
```
