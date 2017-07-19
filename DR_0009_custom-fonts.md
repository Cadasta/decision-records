# 0009: Supported Project Data Fonts in the Cadasta Platform

- **Author:** Chandra Lash
- **Status:** WIP Draft Proposal, 2017-07-11
- **Backlog Issue:** [#62](https://github.com/Cadasta/cadasta-docs/issues/62)



## Introduction

As our platform grows and new partner demands increase, we need a policy for handling custom fonts to support project data, particular non-Latin script fonts. 


## Problem overview

Our partners can choose to collect project data in numerous languages. If text characters are requested by the browser but can not be displayed because there is no available font support, small boxes are shown to represent the characters. In slang those small boxes have sometimes been called "tofu". 


## Suggested solution

- The Cadasta Platform will only support Unicode fonts with [UTF-8 encoding](https://en.wikipedia.org/wiki/UTF-8).
- It is our standard practice to rely on the user to have fonts locally available.
- Cadasta should add a supported font section to our platform documention providing best practice advice for accessing project data with uncommon fonts.


## Other considerations

- Customs fonts will be available in the platform on a case-by-case basis. As of July 2017, the script fonts KNU, Padauk and Noto Benghali fonts are available.
- As [Google Noto Fonts](https://www.google.com/get/noto/#/family/noto-sans-hans) become more complete, we could implement importing specific fonts as a font-face and then use multiple unicode-ranges. We would need to ensure this is done in such a way as not to impede performance.


## Further reading

- [UTF-8 and Unicode](http://www.utf-8.com/)
- [Wikipedia Universal Language Selector/Webfonts](https://www.mediawiki.org/wiki/Universal_Language_Selector/WebFonts)
- [Google Noto Fonts: Here's What No Tofu Means](http://www.techtimes.com/articles/181674/20161012/google-noto-fonts-heres-what-no-tofu-means.htm)
- [Web Font Optimization](https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/webfont-optimization)

