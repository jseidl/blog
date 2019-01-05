---
title: Enumerating Android installed applications without special permissions
author: Jan Seidl
type: post
date: 2013-04-30T00:14:28+00:00
url: /posts/enumerating-android-installed-applications-without-special-permissions/
categories:
  - Mobile
tags:
  - android
  - enumeration
  - mobile

---
Sometimes I took some random Android app I&#8217;ve recently installed on my phone and start performing some tests on it. It&#8217;s not uncommon to see unauthenticated <acronym title="Application Programming Interface">API</acronym> requests, plaintext <acronym title="HyperText Transfer Protocol">HTTP</acronym> communication and some obfuscation logic or hardcoded credentials rammed down at client code. As I use to say in my <acronym title="Supervisory Acquisition and Data Control">SCADA</acronym> talks, 

> &#8220;everything that is &#8216;new&#8217; is prone to &#8216;newbie&#8217; mistakes&#8221;

This is not different for mobile development. The urge for delivery and the hype of start-ups and its fast-paced deployment philosophy usually results on developments with little to no attention to security.

Well, today I was reversing one of those applications. I won&#8217;t disclose the name of it for obvious reasons so don&#8217;t ask. I started simply by jacking up [ProxyDroid][1] to [Burp][2] and started to use the app normally.

Then I saw something on Burp that made me **REALLY** concerned. 

{{< highlight text >}}
POST /api/userCompetitors?uuid=MY_ANDROID_ID&competitors=COMPETITOR_APP_I_HAD_INSTALLED
{{< /highlight >}}

**That prick was spying on me!** So I got a little mad and decided to rip the `apk` open and take a look at the code to see how that was being done.

As I rolled my eyes through the code seeking the &#8220;competitor mining&#8221; code, I was getting some usual juice like finding that all of the requests were unauthenticated that could lead to almost complete data extraction from partners, customers and transactions, scary! 

**And then I found it!** Consider the following snippet from the app:

{{< highlight java >}}
private static final Map&lt;String, String> competitors = new HashMap();

  static
  {
    competitors.put("br.com.REMOVED", "REMOVED");
    competitors.put("gr.REMOVED.REMOVED", "REMOVED");
    competitors.put("br.REMOVED.com", "REMOVED");
    // ... edited ...
  }
  public static List&lt;String> getCompetitors()
  {
    ArrayList localArrayList = new ArrayList();
    try
    {
      // Get Installed Packages?? I don't remember anything about this on the "Permissions" tab
      Iterator localIterator = getInstalledPackages().iterator(); 
      while (localIterator.hasNext())
      {
        String str1 = (String)localIterator.next();
        String str2 = (String)competitors.get(str1);
        if (str2 != null)
        {
          Log.d(TAG, "Found competitor: " + str2);
          localArrayList.add(str2);
        }
      }
    }
    catch (Exception localException)
    {
      Log.e(TAG, "Could not get competitors", localException);
    }
    return localArrayList;
  }
{{< /highlight >}}

So it seems that any app can call `getInstalledPackages` from the [PackageManager][3] class on Android <acronym title="Software Development Kit">SDK</acronym> and enumerate all you installed apps? Yes, [this is true][4].

**And there&#8217;s more!** According to the documentation:

> A List of PackageInfo objects, one for each package that is installed on the device. In the unlikely case of there being no installed packages, an empty list is returned. **If flag GET\_UNINSTALLED\_PACKAGES is set, a list of all applications including those deleted with DONT\_DELETE\_DATA (partially installed apps with data directory) will be returned**.

So any app can also return apps you uninstalled!

## But Jan, how bad is this?

Well, basically I don&#8217;t like anything tracking me and that&#8217;s why I make extensive use of privacy tools when I&#8217;m online, so I don&#8217;t like either any app phoning-home with the list of apps I have. 

In the case above, this company was using this in order to collect market data from competitors and payments gateway. As many may think this is &#8220;part of the business&#8221; I think this is not also wrong and offending as this capability of the Android System really should have its own permission.

For me, as [there is no system-defined permission for reading installed apps][5], this is clearly a case of _&#8220;it&#8217;s not a bug, it&#8217;s a feature&#8221;_ that is really creepy.

## UPDATE: Comments from Zigurd on Hacker News

I&#8217;ve submitted this post to [Hacker News][6] and an [user raised some interesting questions][7] that I&#8217;ll post below and comment on them:

### Package names weren&#8217;t made to be private

> There are a few rationales one could cite for this:
  
> One could probably obtain the package names of all the apps out there, so, as long as package names can be used to access components in Android, this is information that could be extracted by trial and error anyway.

I totally agree on &#8220;package names weren&#8217;t designed to be private&#8221;. As I cited this is a feature not a bug but I think it should. Not that the package name should be a secret but you shouldn&#8217;t be able to retrieve the installed ones from the user system.

### The impact doesn&#8217;t pose a threat

> On the other hand, everything you can do with this knowledge is controlled by sandboxing and permissions, so having this knowledge doesn&#8217;t give you anything beyond what you could have with a good guess.
  
> Lastly, one would have to reinvent package naming around names that cannot be guessed. To sum it up, package names weren&#8217;t designed to be private, and retrofitting privacy to package names is hard.

I disagree with &#8220;everything you can do with this knowledge is controlled by sandboxing and permissions, so having this knowledge doesn&#8217;t give you anything beyond what you could have with a good guess.&#8221; because this information can be used to later push application-specific ads or even try to present the user with data in order to make him click on an ad or link and get exposed.

### There are several ways on enumerating (guessing) installed packages

> I agree there is a privacy concern, however, if you took away that method in the package manager, you could still try invoking, say, standard methods in components in packages and guess their names.
  
> The developers&#8217; domain names are public, so there is no way to prevent guessing parts of package names in Android and probably no way to prevent guessing complete package names.
  
> Let&#8217;s take a use case: I want to secretly check if you have banking apps installed. I can install them and discover their package names. Then I can make a malicious app that checks if some component of those apps exists, by checking for an intent filter match, for example. Then I present you with a targeted phishing attack that looks like those apps&#8217; screens. You didn&#8217;t need to enumerate all installed packages to do that. 

I get this point and I do agree that are diferrent ways on enumerating apps **but you need to do some guessing**. The evil part I think is that you can enumerate ALL apps _without_ need to guessing.

### There are legitimate uses for enumerating all the packages

> There are legitimate uses for enumerating all the packages. I&#8217;ve used it for a plug-in architecture for an app that enables 3rd party plug-ins. 

And I also agree with you that there are legitimate uses for that, as you have for sending/receiving SMS, but my point is: Users should be warned that this app is attempting to do that so they can judge where to install it or no, as you have for all other sensitive information.

I&#8217;m really happy on having this article bringing up this discussion because it&#8217;s an example of how even to minor issues, the security concerns should be evaluated.

 [1]: https://play.google.com/store/apps/details?id=org.proxydroid
 [2]: http://portswigger.net/burp/proxy.html
 [3]: https://developer.android.com/reference/android/content/pm/PackageManager.html
 [4]: https://developer.android.com/reference/android/content/pm/PackageManager.html#getInstalledPackages(int)
 [5]: https://developer.android.com/reference/android/Manifest.permission.html
 [6]: https://news.ycombinator.com/
 [7]: https://news.ycombinator.com/item?id=5631935
