# Vue CookieConsent Component

This this is a Vue Single File Component for cookie consent management. 

- Saves consent status to local storage 
- Drops analytical and functional cookies based on user consent

## Dev dependencies
- postcss-nested
- tailwindcss
- autoprefixer
 

## Dependencies
- VueGtag
- TailwindCSS

###Cookie category id’s
- Functional cookies: 1
- Analytical cookies: 2

## Usage

1. Import component into Vue app
`import CookieConsent from '~/components/shared/CookieConsent'`

2. Register the imported component locally within components object
```
components: {
    CookieConsent
}
```
3. Output the component in the template
`<CookieConsent  />`

You can pass in properties like this:
`<CookieConsent  :privacyPolicyUrl=""  />`


## Browser support

Works in all modern browsers and IE11

Will add further detailed instructions when I get time. 

For general documentation on Single File Components, see VueJs guide: https://vuejs.org/v2/guide/single-file-components.html





