## Contents

- [x] Debugging your JavaScript running on other domains.
- [x] Running Custom JavaScript on other domains. (Include console approach and tell the execution of JS)
- [x] Modifying Query Parameters
- [x] Writing Front End Code dependent on API even if they are not in place.
- [] Modifying Request and Response Headers
- [x] Ensuring third party libraries do not block your website.
- [] Modifying and Blocking cookies on some Domain.
- [] Removing Iframe security headers.
- [] Tips and Tricks (Bookmarks, Block Time Killing Websites)

## Audience
This Article will be most beneficial to Web Developers using Google Chrome. Other readers can enjoy different techniques and the approaches you can use to solve similar issues.

## Article

A Web Developer spends more time with a web browser than anything else in this world. Web Developers often find themselves in a situation when they want more control over the sites being loaded in their browser. Loading a web page in browser includes HTTP(s) request from the browser and corresponding HTTP(s) response from the web server. Having control over HTTP(s) request and response makes Web Developers more powerful and let them do what they want with the page.

In this article, I will be sharing few tips and tricks of HTTP Request Manipulation.

1. Debug your javascript code running on Customer's website.
2. Execute some Custom Javascript code on a webpage.
3. Modify Query Parameters in a URL.
4. Work on front end Code when APIs are not in place.
5. Ensure the impact of slow (or no) loading of external script on your webpage.
6. Block a cookie for a particular domain.
7. Bypass Iframe Security headers for a particular website.

In the end I will wrap up with some tricks I use everyday.

### Tip 1: Debug your javascript code running on Customer's website.

#### Scenario
Suppose you have created a script to Insert Share buttons on a webpage. Users downloaded your javascript, hosted it on their server and included on their webpage. Consider your script is throwing some errors on a particular User website and you manage to have a fix ready for it. Since your script is running on other domain which obviously you can not control, how would you verify that your fix will run perfectly next time User performs the same steps (Download latest version, Host the latest and include on webpage).

#### Trick A: Redirect URL Mechanism
All you need is to redirect the URL of script present on website with the URL of script on your localhost where you made the fix.

#### Tools: There are many Browser Extensions using which you can redirect an HTTP(s) URL.

1. Chrome Browser: [Requestly](chrome.google.com/webstore/detail/requestly/mdnleldcmiljblolnjhpnblkcekpdkpa),
[Switcheroo Redirector](https://chrome.google.com/webstore/detail/switcheroo-redirector/cnmciclhnghalnpfhhleggldniplelbg?hl=en)

2. Firefox browser: [Redirector](https://addons.mozilla.org/en-us/firefox/addon/redirector/) (But it only works till certain Firefox version)

3. Windows/Mac Platform: Charles Proxy and Fiddler.

#### Example: Using Requestly Chrome Extension

![Requestly Redirect Url Example](http://codebrackets.github.io/articles/01-http-request-debugging/images/requestly_redirect_url_example.png)
 
#### Note: 
If you redirect an HTTPS script to HTTP script on a secure website then your browser will block the http script with an indicator in Address bar. You need to Enable [Mixed Content](http://www.howtogeek.com/181911/htg-explains-what-exactly-is-a-mixed-content-warning/) to load http script on secure pages.

There are lots of scenarios where you need "Redirect" approach without having permissions to modify the actual website. 

### Tip 2: Running Custom Javascript on a website.

#### Scenario 
Suppose you created a script to track User clicks on the page and show reports corresponding to the clicks. Now, you want to demonstrate how your Javascript code works on website (www.customer.com). Your script is obviously not present on www.customer.com.  

#### Trick A: Using Browser Developer Tool

All modern browsers provide Developer tools with rich set of capabilities. Using Developer Tool, you can execute your Javascript code in the context of the webpage opened in the tab. 

Suppose I want to extract the username on [my profile tab of StackOverflow Page](http://stackoverflow.com/users/816213/blunderboy?tab=profile), I can execute the following code in DevTools Console

    function isJqueryPresentOnPage() { 
      return typeof window.jQuery !== 'defined' && typeof window.jQuery.fn.jquery !== 'undefined';
    }
    isJqueryPresentOnPage();
    
    Output: true
    
    function getJqueryVersion() { 
      return jQuery.fn.jquery 
    }
    getJqueryVersion()
    
    Output: "1.7.1"
    
    function getUserProfileName() { 
      return $('#user-card').find('.user-card-name').html().trim().split(' ')[0] 
    }
    getUserProfileName();
    
    Output: "blunderboy"

##### Have a look at this snapshot: 

![Stackoverflow Profile Page Code Execution](http://codebrackets.github.io/articles/01-http-request-debugging/images/StackOverlow_Code_execution.png)

This approach is easy but it comes with certain limitations. Couple of points should be observed.
1. Code is executed after all the scripts have been executed and DOM is also loaded
2. Any modification done in DOM by this code would not persist. As soon as you reload the page you need to execute the same script again to get back those changes.

This is a handy approach if you just need to calculate or read some values.

#### Trick B: Using Redirect URL mechanism

We can look at the page source to identity one or more third party scripts like Analytics Script or Share  Buttons script or Disqus Comment box script. Once we have identified any such script which does not impact the functionality of the webpage, we can redirect that script to our script hosted somewhere (May be localhost). Now, whenever the page is loaded, third party script will be replaced by our script and our script executes every time the page is loaded.

This approach is very handy while giving demo to customers who have not bought your product yet. This is really impactful and Customers gain more attention when you demonstrate your product on their website LIVE without involving any minimal effort of integration.

### Tip 3: Modifying Query Parameters in HTTP Request URL

#### Scenario
Suppose your website's behavior is affected by the query parameters present in the URL. For example, Adding debug=true query parameter to URL starts printing logs in the browser console as User takes some actions. Now you want to add this query Parameter always as you navigate pages of your website.

#### Trick A: Manually modifying/adding `debug=true` parameter and reloading the page.
This trick works good if you don't need to navigate to any other page. 

#### Trick B: Replace in URL Mechanism
If URL already contains a query parameter say productId=1345 and you want to add debug parameter to it.
You can achieve this via [Requestly](https://chrome.google.com/webstore/detail/requestly/mdnleldcmiljblolnjhpnblkcekpdkpa) like this:

![Requestly Replace URL Example to Modify Query Parameter](http://codebrackets.github.io/articles/01-http-request-debugging/images/Replace-Rule-Example.png)

### Tip 4: Write Front End Code when APIs are not in place 

#### Scenario
In Single Page App Development we fetch the data as we need it. For Example: Suppose you have build a page where you show a list of most favorited tweets of the day. The number of tweets is pretty large so it can not be loaded in one request. Hence, you have implemented Infinite Scrolling mechanism but you still lack the backend API hence you can not ensure your code works to which extent.

#### Trick A: Make changes in your code where you make the GET call an return dummy JSON response.
For Example: In case of Backbone framework, you can modify Model/Collection `parse` method to return dummy response. 
Problem with this approach is neither you can commit your code in such a state nor you will be able to deliver to QE team for verification. In simple words, it is incomplete and as it is said 

> If a code is not tested, it does not work.

#### Trick B: Use Mock API Service
You can mock your API response on a mock service like [Mockable.io](https://www.mockable.io). You might be thinking that you still need to change your end point in the code to point to mockable service URL. Wait! I won't let you do such a thing again.

You can use Requestly "Replace in URL" mechanism to replace your end points with mockable URLs like

![Requestly Replace Domain Example](http://codebrackets.github.io/articles/01-http-request-debugging/images/Requestly-Replace-Domain-Example.png)

#### Note: 
You need to add Access-Control-Allow-Origin Header to the response given by the Mockable.io API so ensure your browser does not block [CORS request](https://developer.mozilla.org/en-US/docs/Web/HTTP/Access_control_CORS).

### Tip 5: Ensure the impact of slow (or no) loading of external script on your page.

#### Scenario
Suppose your website is www.some-online-retailer.com and you included an external script http://cdn.exteral-service.com/js/share-buttons.js. Since this is an external server, you do not have any control over load time of this script. But You must ensure that if load time of this script is high or this script does not load at all, your page would load fine. 

#### Trick A: Throttling Mechanism in Charles Proxy
Use [Charles proxy](http://www.charlesproxy.com/documentation/proxying/throttling/) to Implement Throttling. From documentation of Charles Throttling page:

> Charles can be used to adjust the bandwidth and latency of your Internet connection. This enables you to simulate modem conditions using your high-speed connection.
> The bandwidth may be throttled to any arbitrary bytes per second. This enables any connection speed to be simulated.

#### Trick B: Block URL mechanism in Requestly
You can use Requestly block URL mechanism in order to block certain scripts. Now you can ensure whether your site loads fine or not when call to external script gets failed.

#### Example: Requestly - Block URL Mechanism
![Requestly Cancel Rule Example](http://codebrackets.github.io/articles/01-http-request-debugging/images/Requestly-Cancel-Rule-Example.png)
