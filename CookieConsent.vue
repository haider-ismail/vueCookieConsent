<template>
  <div>
    <div v-if="isOpen" class="cookieConsent w-full bg-grey-lighter py-3 lg:py-4 shadow-lg">
      <!-- Cookie consent banner -->
      <div class="flex flex-wrap container mx-auto items-center">
          <div class="w-full lg:flex-1">
            <slot name="message">
              <span class="mb-2 block text-sm">{{ bannerText }}</span>
              <span v-if="privacyPolicyUrl" class="block mt-0 mb-4 lg:mb-0 block text-sm">To find out more about our use of cookies, please see our
              <a  class="cookie__link text-primary" target="_blank" :href="privacyPolicyUrl">Privacy Policy</a>.</span>
            </slot>
          </div>
          <div class="flex-shrink-0 flex -mx-1">
            <button class="self-center mx-1 bg-green hover:bg-green-dark hover:border-green-dark  text-white button  font-normal inline-block no-underline text-sm uppercase border border-green px-2 xs:px-4 py-2 text-center rounded rounded-full" @click="acceptRecommended">{{ buttonTextAccept }}</button>
            <button class="self-center mx-1 flex-no-shrink hover:bg-primary  hover:text-white hover:border-primary button  font-normal inline-block bg-grey-light text-black no-underline text-sm uppercase border border-grey-light px-2 xs:px-4 py-2 text-center rounded rounded-full" @click="togglePrefModal()">{{ buttonTextManage }}</button>
          </div>
      </div>
    </div>
    <!-- ./Cookie consent banner -->

    <!-- Cookie preference modal -->
    <div v-if="prefManagerOpen" class="cookie-pref-modal fixed h-full pin-l pin-t w-full z-50">
      <div class="bg-white cookie-pref-modal__inner max-h-screen overflow-y-scroll max-w-sm md:max-w-md p-6 sm:p-8 w-5/6 sm:w-full">
        <svg aria-label="Close" title="Close" class="w-8 h-8 absolute pin-r pin-t m-2 text-black hover:text-red cursor-pointer" @click="prefManagerOpen = false" fill="none" viewBox="0 0 24 24" stroke="currentColor">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"/>
        </svg>

        <h4 class="leading-normal pr-2 sm:pr-0 mb-3">Manage Cookie preferences</h4>
        <p class="text-sm mt-0 mb-4 leading-normal">Your changes will take affect when you click "Save &amp; exit" or on next page reload.</p>
        <p v-if="privacyPolicyUrl" class="text-sm mt-0 mb-6 leading-normal">To find out more about our use of cookies, please see our <a target="_blank" :href="privacyPolicyUrl">Privacy Policy</a>.</p>

        <div class="flex items-center mb-4">
          <label class="switch">
            <input id="functionalCookiesToggle" type="checkbox" v-model="functionalCookiesConsent">
            <span class="slider round"></span>
          </label>
          <label class="text-sm ml-3" for="functionalCookiesToggle">Functional cookies</label>
        </div>

        <div class="flex items-center">
          <label class="switch">
            <input id="analyticsToggle" type="checkbox" v-model="analyticalCookiesConsent">
            <span class="slider round"></span>
          </label>
          <label class="text-sm ml-3" for="analyticsToggle">Analytical cookies</label>
        </div>

        <div class="flex -mx-2 mt-8 items-center flex-wrap">
          <button @click="reloadPage()" class="mb-3 sm:mb-0 w-full sm:w-auto mx-2 self-center mx-1 flex-shrink-0 bg-primary hover:bg-primary-dark hover:border-primary-dark  text-white button  font-normal inline-block no-underline text-sm  border border-primary px-4 py-2 text-center rounded rounded-full">Save &amp; exit</button>
          <button class="mb-3 sm:mb-0 w-full sm:w-auto mx-2 self-center mx-1 flex-shrink-0 bg-green hover:bg-green-dark hover:border-green-dark  text-white button  font-normal inline-block no-underline text-sm  border border-green px-4 py-2 text-center rounded rounded-full" @click="acceptRecommended">{{ buttonTextAccept }}</button>
          <button @click="denyAll()" class="w-full sm:w-auto text-sm text-primary mx-2">{{ buttonTextRejects }}</button>
        </div>

      </div>
    </div>
    <!-- ./Cookie preference modal -->
  </div>
</template>

<script>
import Vue from 'vue'
import VueGtag from 'vue-gtag'

/**
 * @author: Haider Ismail
 * Usage guide: https://github.com/haider-ismail/vueCookieConsent
 */

export default {
  name: 'CookieConsent',
  props: {
    buttonTextAccept: {
      type: String,
      default: 'Accept all'
    },
    buttonTextReject: {
      type: String,
      default: 'Reject all'
    },
    buttonTextManage: {
      type: String,
      default: 'Manage settings'
    },
    bannerText: {
      type: String,
      default: 'We use cookies to improve user experience and analyse website traffic. Click "Accept" to allow cookies on this website.'
    },
    privacyPolicyUrl: {
      type: String,
      default: 'www.example.com/privacy' //@NOTE: Update to point to where the privacy policy sits (e.g. webpage or PDF)
    }
    position: {
      type: String,
      default: 'bottom'
    },
    gaId: {
      type: String,
      default: '000000' //@NOTE: Update default GA ID or pass in via props
    },
    leadForensicId: {
      type: String,
      default: '000000' //@NOTE: Update default lead forensic ID or pass in via props
    }
  },
  data () {
    return {
      isOpen: false,
      prefManagerOpen: false,
      analyticalCookiesConsent: (typeof window !== 'undefined' ? this.getAnalyticalCookiesConsentStatus() : false),
      functionalCookiesConsent: (typeof window !== 'undefined' ? this.getFunctionalCookiesConsentStatus() : false)
    }
  },
  computed: {
    containerPosition () {
      return `cookie--${this.position}`
    }
  },
  mounted () {
    this.$nextTick(() => {
      if (process.browser) {
        if (!this.getGDPR()) {
          this.isOpen = true
        }

        if (this.getAnalyticalCookiesConsentStatus()) {
          this.runAnalyticalScripts()
        }

        if (this.getFunctionalCookiesConsentStatus()) {
          this.runFunctionalScripts()
        }

        this.$root.$on('prefModalToggled', () => {
          this.togglePrefModal()
        })
      }
    })
  },
  methods: {
    getEnabledCookieCategories () {
      let categories = []

      if (this.getFunctionalCookiesConsentStatus()) {
        categories.push('1')
      }
      if (this.getAnalyticalCookiesConsentStatus()) {
        categories.push('2')
      }
      return categories
    },
    getGDPR () {
      if (process.browser) {
        return Boolean(localStorage.getItem('GDPR:accepted', true))
      }
    },

    /**
     * This sets the defined recommended cookie categories (analytical and functional)
     */
    acceptRecommended () {
      this.accept(['1', '2'])
      this.reloadPage()
    },

    accept (cookieTypes = []) {
      if (process.browser) {
        if (!cookieTypes.length) return

        if (this.getEnabledCookieCategories().length) {
          let catsToAdd = []
          const existingCatIds = this.getEnabledCookieCategories()

          cookieTypes.forEach(categoryID => {
            if (existingCatIds.filter(catId => catId === categoryID).length === 0) {
              catsToAdd.push(categoryID)
            }
          })

          const acceptedCookieIds = existingCatIds.concat(catsToAdd)
          localStorage.setItem('GDPR:accepted', acceptedCookieIds)
        } else {
          localStorage.setItem('GDPR:accepted', cookieTypes)
        }

        cookieTypes.forEach(categoryID => {
          switch (categoryID) {
            case '1':
              this.functionalCookiesConsent = true
              this.runFunctionalScripts()
              break
            case '2':
              this.analyticalCookiesConsent = true
              this.runAnalyticalScripts()
              break
            default:
              break
          }
        })

        this.isOpen = false
      }
    },
    denyAll () {
      if (process.browser) {
        this.isOpen = false
        localStorage.setItem('GDPR:accepted', false)
        this.analyticalCookiesConsent = this.getAnalyticalCookiesConsentStatus()
        this.functionalCookiesConsent = this.getFunctionalCookiesConsentStatus()

        if (this.prefManagerOpen) {
          this.prefManagerOpen = false
        }
      }
    },
    deny (cookieType = null) {
      if (process.browser) {
        if (cookieType === null) return

        if (this.getEnabledCookieCategories().length) {
          const existingCatIds = this.getEnabledCookieCategories()
          const acceptedCookieIds = existingCatIds.filter(catId => catId !== cookieType)

          if (acceptedCookieIds.length) {
            localStorage.setItem('GDPR:accepted', acceptedCookieIds)
          } else {
            localStorage.setItem('GDPR:accepted', false)
          }
        }

        switch (cookieType) {
          case '1':
            this.functionalCookiesConsent = false
            break
          case '2':
            this.analyticalCookiesConsent = false
            break
          default:
            break
        }
      }
    },
    runAnalyticalScripts () {
      //@NOTE: run all analytical scripts within this method
      console.log('[runAnalyticalScripts] -->')
      if (this.gaId) {
        this.createGAScript()
      }

      if (this.leadForensicId) {
        this.createLeadForensicsScript()
      }
    },
    runFunctionalScripts () {
      //@NOTE: run all functiona; scripts within this method
    },
    createGAScript() {
      Vue.use(VueGtag, {
        config: {
          id: this.gaId,
          params: {
            send_page_view: true
          }
        }
      })
    },
    createLeadForensicsScript () {
      console.log('[createLeadForensicsScript] -->')
      // Dynamically add LeadForensic script
      const head = document.querySelector('head')
      const firstScript = document.getElementsByTagName('script')[0]
      const leadForensicsScript = document.createElement('script')
      leadForensicsScript.src = `https://secure.leadforensics.com/js/${this.leadForensicId}.js`
      head.insertBefore(leadForensicsScript, firstScript)
    },
    togglePrefModal () {
      this.prefManagerOpen = !this.prefManagerOpen
    },
    getFunctionalCookiesConsentStatus () {
      const categories = localStorage.getItem('GDPR:accepted')
      if (categories !== null && categories.length) {
        const categoriesArray = categories.split(',')
        return (categoriesArray.indexOf('1') >= 0)
      }
      return false
    },

    getAnalyticalCookiesConsentStatus () {
      const categories = localStorage.getItem('GDPR:accepted')
      if (categories !== null && categories.length) {
        const categoriesArray = categories.split(',')
        return (categoriesArray.indexOf('2') >= 0)
      }
      return false
    },
    removeAnalyticCookies () {
      document.cookie = `_gid= ; domain=${window.location.host.replace('www.', '.')}; path =/; expires = Thu, 01 Jan 1970 00:00:00 GMT`
      document.cookie = `_ga= ; domain=${window.location.host.replace('www.', '.')}; path =/; expires = Thu, 01 Jan 1970 00:00:00 GMT`
      document.cookie = `_gat_gtag_${this.gaId}= ; domain=${window.location.host.replace('www.', '.')}; path =/; expires = Thu, 01 Jan 1970 00:00:00 GMT`
      document.cookie = `lfuuid= ; expires = Thu, 01 Jan 1970 00:00:00 GMT;`
    },
    removeLeadForensicsCookies () {
      document.cookie = `lfuuid= ; expires = Thu, 01 Jan 1970 00:00:00 GMT;`
    },
    reloadPage() {
      window.location.reload()
    }
  },
  watch: {
    analyticalCookiesConsent (consent) {
      if (!consent) {
        this.deny('2')
        // Remove added cookies
        if (this.gaId) {
          this.removeAnalyticCookies()
        }

        if (this.leadForensicId) {
          this.removeLeadForensicsCookies()
        }

        console.log('%c [analyticalCookiesConsent] --> disable analytical cookies', 'color: red')

        return
      }

      this.accept(['2'])
      console.log('%c [analyticalCookiesConsent] --> enable analytical cookies', 'color: green')
    },
    functionalCookiesConsent (consent) {
      if (!consent) {
        this.deny('1')
        // Remove added cookies

        console.log('%c [functionalCookiesConsent] --> disable functional cookies', 'color: red')
        return
      }

      this.accept(['1'])
      console.log('%c [functionalCookiesConsent] --> enable functional cookies', 'color: green')
    }
  }
}
</script>

<style scoped>
  .cookieConsent {
    z-index: 50;
    position: fixed;
    bottom: 0;
    left: 0;

    &__link {
      text-decoration: underline;
      transition: all .25s;

      &:hover {
        text-decoration: none
      }
    }

  }

  .cookie-pref-modal {
    &:before {
      content: '';
      background: rgba(0,0,0,0.5);
      @apply block w-full h-full pin-t pin-l;
    }

    &__inner {
      position: absolute;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
    }
  }

  .switch {
    position: relative;
    display: inline-block;
    width: 46px;
    height: 26px;

    input[type="checkbox"] {
      opacity: 0;
      width: 0;
      height: 0;

      &:checked + .slider {
        background-color: #38c172;
      }

      &:focus + .slider {
        box-shadow: 0 0 1px #38c172;
      }

      &:checked + .slider:before {
        -webkit-transform: translateX(18px);
        -ms-transform: translateX(18px);
        transform: translateX(18px);
      }
    }
  }

  .slider {
    position: absolute;
    cursor: pointer;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background-color: #ccc;
    -webkit-transition: .4s;
    transition: .4s;

    &:before {
      position: absolute;
      content: "";
      height: 20px;
      width: 20px;
      left: 4px;
      bottom: 3px;
      background-color: white;
      -webkit-transition: .4s;
      transition: .4s;
    }

    &.round {
      border-radius: 34px;
      &:before {
        border-radius: 50%;
      }
    }
  }
</style>
