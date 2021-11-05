# Vue CookieConsent Component

This this is a Vue Single File Component for cookie consent management. 

- Saves consent status to local storage 
- Drops analytical and functional cookies based on user consent

To minimise the number of dependencies required, I have intentionally not included Typescript in this Single File Component. If your project is using Typescript, I highly recommmend you make the required updates to the Javascript methods to leverage it.

## Dev dependencies
- postcss-nested
- tailwindcss
- autoprefixer
 

## Dependencies
- VueGtag
- TailwindCSS

### Cookie category id’s
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

### Conditional props
#### buttonTextAccept 
type: `String`  
default: 'Accept all' 

#### buttonTextReject
type: `String`  
default: 'Reject all' 

#### buttonTextManage  
type: `String`  
default: 'Manage settings'  

#### bannerText 
type: `String`   
default: 'We use cookies to improve user experience and analyse website traffic. Click "Accept" to allow cookies on this website.' 

#### privacyPolicyUrl 
type: `String`  
default: 'www.example.com/privacy' //@NOTE: Update to point to where the privacy policy sits (e.g. webpage or PDF)  

#### position 
type: `String`   
default: 'bottom'   

#### gaId  
type: `String`  
default: '000000' //@NOTE: Update default ID or pass in via props  

#### leadForensicId  
type: `String`  
default: '000000' //@NOTE: Update default ID or pass in via props  

## Browser support

Works in all modern browsers and IE11

Will add further detailed instructions when I get time. 

For general documentation on Single File Components, see VueJs guide: https://vuejs.org/v2/guide/single-file-components.html





