Brave Privacy Preserving Ads - DRAFT

Version: 0.7
Published: January 27, 2016

## Introduction

Much of the content on the internet today is funded through online advertising. While free access to this content is useful and beneficial, the methods used to target and deliver the advertisements are less than ideal both in terms of privacy and performance. The purpose of this document is to outline a specification to enable online advertising that combines effective targeting with high levels of privacy and low page overhead.

## Terminology

- *Advertiser* - The entity that "owns" the ad and wants to track its effectiveness either directly or through a proxy
- *Publisher* - A provider of content (news site, blog, etc...)
- *Browser* - The software that retreives and displays the content
- *Device* - The hardware/software the Browser runs on
- *User* - The actual person who uses the Browser on one or more Devices
- *Ad Service* - The service the Browser contacts to retreive privacy-safe ads (Brave)

Definitions of related terms used in this document:

- *PII* - Personally identifiable information. This would include name, address, email, etc...
- *User Data* - Other data connected to the user that would not be considered PII. This includes information such as IP address, persistent identifiers associated with a user, browser or device and browsing history.
- *Cookie* - A method of persisting data on the browser that can be read and updated by the domain that set it
- *Fingerprinting* - Identifying a unique Browser by methods other than persistent data storage (cookies, etc...)
- *Ad* - The content delivered from the Advertiser to be displayed in the Browser
- *Campaign* - One or more ads that the Advertiser has associated with a goal
- *Ad Tag* - The javascript/html code that will be executed in the browser to display the Ad
- *Creative* - The viewiable portion of the Ad (image, etc...)
- *Placement* - An available Ad "slot" on a publisher site
- *Impression* - An actual ad view by a user
- *Landing Page* - The destination URL you are taken to when clicking on an Ad
- *Click Pixel* - A link that redirects to the intended landing page and logs the association between the user, the Ad and the Landing Page
- *Tracking Pixel* - Usually a script or non-visible image tag that records your browsing activity wherever it is present. Tracking pixels can be placed in ads or directly in publisher or advertiser sites
- *Conversion* - The completion of some advertiser-defined activity (ad click, online purchase, etc...) for which the displayer of the ad should be credited
- *Attribution* - The process of determing who gets the credit (payment) for the display of an Ad or the completion of a Conversion
- *Segment* - Also known as "audience segment", this is a category used to define interest, intent to purchase or demographic information typically captured or inferred from tracking pixels or collected from user supplied data
- *Pseudo-Identier* - A set of attributes that are unique enough to act as a proxy for a user identifier. For example: zipcode, date of birth and gender typically match with at most 3 or 4 people in the United States

## Privacy Preserving Ads

#### Goals

- Protect user privacy
- Improve ad relevancy and timeliness
- Reduce the Ad "footprint" (bandwidth, page placement, page load blocking, etc...)
- Provide user control and opt-out for ad content

#### Requirements

- Prevent access to any PII
- Expose as little user data as possible
- No persistent user identifiers
- No 3rd-party tracking
- No storage of 3rd-party data on the Browser (cookies or any other local storage mechanism)

#### Segment Modeling

Publishers should be rewarded for producing high-quality content and those rewards are determined by the amount of money that Advertisers are willing to pay to reach a particular audience. Reaching as much of the target audience as possible while limiting false positives generally means that advertisers will be willing to pay more per view (or conversion). To provide the highest level of privacy, segment modeling is done entirely within the browser. There are advantages and disadvantages to this approach. The primary advantage is the amount of data the browser has access to: complete browsing history, time spent on pages, web searches, social media, etc... All of this data is readily available within the Browser without sacrificing privacy. The primary disadvantage is that you are limited to a sample size of one. We will explore methods for including privacy-safe aggregated data in the future.

#### Ad Serving

Now that we have established the basis for privacy-safe mapping of Users to Segments, the next step is to actually deliver the relevant Ads to the Browser based on those Segments. Other methods have been proposed for delivering ads without exposing any Segment data. These methods generally involve sending several ads to the Browser that are targeted at differnt segments. The Browser looks for a matching ad based on the Segment and displays that ad to the User. We believe this approach is impracticle. Advertisers are often looking to reach a very specific audience (interested in a specific model car for instance) in a particular region. While that population will be large enough to provide reasonable levels of privacy, there could be hundreds of similar campaigns running simultaneously. The number of ads that would be required to provide a high likelihood of a match would be very large.

On the opposite end of the spectrum we could expose all of the segments on every ad request. This would provide the most opportunities for a match with an Advertiser, but would almost certainly turn the segments into a Psuedo-Identifer that could be used to track the user over time.

The approach used here is somewhere in between. The Browser send a small number of Segments to the Ad Service in an anonymous request and the Ad Service returns Ads matching those Segments. This approach exposes a very small amount of user data in exhange for a large increase in efficiency. To ensure that the Browser doesn't send "worthless" segments to the Ad Service that will result in either no matching Ads or matching Ads with low return to the Publisher, the Ad Service will publish a list of segments that Advertisers are currently interested in along with their relative values. The Browser will use this list to prioritize the User Segments it send to the Ad Service and avoid wasteful requests.

There are still some cases where the Browser must request more Ads than it will actually use. The Browser allows the User to "downvote" Ads to blacklist an individual Ad, a Brand or an entire Category. The contents of the blacklist are likely to be unique to each User and could be used as a Psuedo-Identifier similar to the complete list of Segments mentioned earlier as an alternate approach. If the Browser sends a Segment to the Ad Service that contains a blacklist entry, the Browser must request additional Ads, either from the same Segment or an alternate Segment, to deal with the possibility of the Ad Service returning an Ad that the User does not want to see.

#### Reporting

The final piece of the puzzle is reporting. This includes basic campaign stats and data necessary for attribution and conversion measurements. We require an alternate method of collecting this data that does not allow the Ad Service or any 3rd party to track or link any data to a User, Browser or Device while still providing sufficient accuracy and fraud protection that is at least on-par with existing methods. To accomplish this the Browser will generate "tokens" that are periodically submitted to the Ad Service through an anonymous channel. The Browser uses information either provided by the Advertiser in the Ad Tag or read from the URL of blocked Tracking Pixels to group related events that will be used for attribution and conversion determinations. The tokens must meet the following set of requirements:

1. They must not be linkable to any User, Device or other persistent identifier
2. They must provide both a total Impression count and a unique User count for any given Campaign
3. They must not be forgeable by another Browser
4. They must not be linkable across Campaigns

These properties are achieved through a combination of cryptography and non-interactive zero-knowledge proofs which will be detailed separately.

#### Problems and Threats

1. It is possible for the both Ad Service and the Advertiser to link the IP address transmitted with the Ad Request to 3rd-party data associated with PII (Facebook logins for instance). This can be addressed with selective use of TOR. The Browser can use TOR for any requests to the Ad Service that are not latency sensitive. Pre-fetching of Ads and Token submission are good candidates for TOR requests.

2. A malicious 3rd-party could generate tokens that do not correspond to actual events. This problem could be solved through secure remote attestation of the Browser code, but there are no practical general-purpose solutions at this time. It is also possible to raise the cost of this type of attack by adding a computationally expensive operation to the generation of the master key used to mint tokens, but the issue is fundamentally smaller than click/impression fraud in traditional online advertising because the master key generation is controlled by the Ad Service and without a new master key it is impossible to send tokens that appear to come from a different user. A statistically unlikely number of submissions from a single user is easily detectable using existing methods.

3. There will be a mismatch between the count of the number of ads requested and the number of ads actually displayed. This can happen for a variety of reasons including the intentional request of more than necessary which is outlined above. This is related to the existing problem of ad visibility and can be dealt with in the same way. Only visible ads will be reported by the Browser to the Ad Service and only visible Ads should be used in payment calculations.

