# Localization Fake Book
Fake book for localization on Apple platforms. Most examples are plundered from [elswhere](https://github.com/hamsterdam/LocalizationFakeBook#sources).

## NSLocalizedString
Use these macros to take advantage of [genstrings](https://developer.apple.com/library/mac/documentation/Darwin/Reference/ManPages/man1/genstrings.1.html):

- `NSLocalizedString`
- `NSLocalizedStringFromTable`
- `NSLocalizedStringFromTableInBundle`
- `NSLocalizedStringWithDefaultValue`

All use NSBundle’s `localizedStringForKey:value:table:`

[genstrings](https://developer.apple.com/library/mac/documentation/Darwin/Reference/ManPages/man1/genstrings.1.html) usage:
```bash
find . -name *.m | xargs genstrings -o en.lproj
```
Use `-o` to specify output directory; “Localizable.strings” is default. Use `-a` to append.

For the localized string   
```objc  
NSLocalizedString(@“session-player.leave-session”, @“Session player leave session”)  
```
the associated strings files has an entry
```
/* Session player leave session */
“session-player.leave-session” = “Leave Session”;  
```
## String Instances
Always define one localizable string per string instance. In other words, never split strings into parts. 

For example, the Fitbit app might show the strings `“You challenged Charlie“` and `“Charlie challenged you”`. You could localize the string `“%@ challenged %@“` and it would work fine for English. However, this may not work for other languages. Additionally, important context is lost for translators. The correct localized strings would be `”%@ challenged you”` and `”You challenged %@“`.
## String Keys
Use an name-spaced approach for string keys such that they are unique for each context. Do not use a base language for keys.

Each key is unique within a strings file. Thus, something that is unique in the base language becomes unique in all other languages, even when inappropriate. For example, in English, “run” can be used as a both a noun and a verb. In other languages, this may not be the case. It may be tempting to have
```objc  
NSLocalizedString(“Run”, nil);  
```
However, this will result in a single entry in the strings file. Instead, do:
```objc
NSLocalizedString(“session.title.run”, nil);
NSLocalizedString(“precession.start.run”, nil);
```
so that the strings files have a key for each context.
## Format Strings 
To localize strings built at runtime:
```objc
[NSString localizedStringWithFormat:NSLocalizedString(@“session.component.reps.%lu out of reps %lu completed”, nil), completedReps, totalReps];
```
A translator can reorganize the format specifiers by referring to them using `%1$lu` and `%2$lu` in the strings file.
## Plural and Gender
Consider the example `@“%lu sessions”`. To appropriately handle plurals in English, there would need to be two different strings: `@“%lu sessions”` for when there are more than one session and `@“%lu session”` for when there is only one session. However, [other languages have different rules](http://www.unicode.org/cldr/charts/latest/supplemental/language_plural_rules.html).

To handle the complexity, we will use the typical `.strings` entry, normally generated like so:
```objc
[NSString localizedStringWithFormat:NSLocalizedString(@“profile.sessions-count.%lu sessions”, nil), completedReps, totalReps];
```
and a `.stringsdict.` `plist` file with the same name:
```xml
<?xml version=“1.0” encoding=“UTF-8”?>
<!DOCTYPE plist PUBLIC “-//Apple//DTD PLIST 1.0//EN” “http://www.apple.com/DTDs/PropertyList-1.0.dtd”>
<plist version=“1.0”>
<dict>
    <key>profile.sessions-count.%lu sessions</key>
    <dict>
        <key>NSStringLocalizedFormatKey</key>
        <string>%lu %#@lu_sessions@</string>
        <key>lu_sessions</key>
        <dict>
            <key>NSStringFormatSpecTypeKey</key>
            <string>NSStringPluralRuleType</string>
            <key>NSStringFormatValueTypeKey</key>
            <string>lu</string>
            <key>one</key>
            <string>%lu session</string>
            <key>other</key>
            <string>%lu sessions</string>
        </dict>
    </dict>
</dict>
</plist>
```

`.stringsdict` files can also be used to implement more complex pluralization rules. Consider some rules for variations of the string `@“%lu of %lu runs”:
```
Completed runs    Total Runs    Output
0                 0+            No runs completed yet
1                 1             One run completed
1                 2+            One of x runs completed
2+                2+            x of y runs completed
```
The `.stringsdict` would be:
```xml
<?xml version=“1.0” encoding=“UTF-8”?>
<!DOCTYPE plist PUBLIC “-//Apple//DTD PLIST 1.0//EN” “http://www.apple.com/DTDs/PropertyList-1.0.dtd”>
<plist version=“1.0”>
<dict>
    <key>scope.%lu out of %lu runs</key>
    <dict>
        <key>NSStringLocalizedFormatKey</key>
        <string>%1$#@lu_completed_runs@</string>
        <key>lu_completed_runs</key>
        <dict>
            <key>NSStringFormatSpecTypeKey</key>
            <string>NSStringPluralRuleType</string>
            <key>NSStringFormatValueTypeKey</key>
            <string>lu</string>
            <key>zero</key>
            <string>No runs completed yet</string>
            <key>one</key>
            <string>One %2$#@lu_total_runs@</string>
            <key>other</key>
            <string>%lu %2$#@lu_total_runs@</string>
        </dict>
        <key>lu_total_runs</key>
        <dict>
            <key>NSStringFormatSpecTypeKey</key>
            <string>NSStringPluralRuleType</string>
            <key>NSStringFormatValueTypeKey</key>
            <string>lu</string>
            <key>one</key>
            <string>run completed</string>
            <key>other</key>
            <string>of %lu runs completed</string>
        </dict>
    </dict>
</dict>
</plist>
```
## Variable Width Rules
Often it makes sense to change text based on available layout width. Use a `.stringsdict` file to do so.
```xml
<key>Motivation</key>
    <dict>
        <key>NSStringVariableWidthRuleType</key>
        <dict>
            <key>20</key>
			<string>Get er done</string>
			<key>50</key>
			<string>Make it happen, you're a champion</string>
		</dict>
	</dict>
```
## Casing
To perform upper and lower casing, use `lowercaseStringWithLocale:` and `uppercaseStringWithLocale:` or the convenience methods `localizedUppercaseString` and `localizedLowercaseString`.
## File Paths
Retrieve localized file paths using `NSURL`:
```objc
NSURL *url = [NSURL fileURLWithPath:@“/Applications/System Preferences.app”];
NSString *name;
[url getResourceValue:&name forKey:NSURLLocalizedTypeDescriptionKey error:NULL];
NSLog(@“localized name: %@“, name);
```
## Numbers
Use `NSNumberFormatter`.

For pre-set styles, use `localizedStringFromNumber:numberStyle`.
## Dates
Use `NSDateFormatter`. 

For pre-set styles, use `localizedStringFromDate:dateStyle:timeStyle:`. 

For custom date and time styles, use `dateFormatFromTemplate:options:locale:`.
## Names
Use `NSPersonNameComponents` and `NSPersonNameComponentsFormatter` to display names.

```swift
let components = NSPersonNameComponents()
components.givenName = "Charlie"
components.familyName = "Mace"
let formatter = NSPersonNameComponentsFormatter()
formatter.style = .Short
formatter.stringFromPersonNameComponents(components)
```
## Formatters
Formatters are not thread safe. Using them from multiple threads is safe, but mutating them is not.

Caching formatters may be a good idea since creating them is a relatively expensive operation.
## Debugging
In user defaults:
- NSDoubleLocalizedStrings
- AppleTextDirection
- NSForceRightToLeftWritingDirection
- NSShowNonLocalizedStrings
- NSShowNonLocalizableStrings
- AppleLanguages
- AppleLocale

To check string files for syntax errors, use [plutil](https://developer.apple.com/library/mac/documentation/Darwin/Reference/ManPages/man1/plutil.1.html). 
## Locale
The systemwide `[NSLocale currentLocale]` might not be the one the app is using. If the user’s system is set to Valyrian, but the app only supports English and Spanish, it will use the default language. Using the systemwide locale with a date formatter will result in a date formatted for that locale but the rest of the app will be in the default language. To use the language chosen by the app:
```objc
NSString *localization = [NSBundle mainBundle].preferredLocalizations.firstObject;
NSLocale *locale = [[NSLocale alloc] initWithLocaleIdentifier:localization];  
```

You can inspect locale instance properties, such as `NSLocaleUsesMetricSystem` using `objectForKey:`.

The classes available for working with locale are:
- `NSLocale`
- `NSDateFormatter`
- `NSNumberFormatter`
- `NSCalendar`
- `NSTimeZone`
- `NSString`

## Current Language
```objc
[[NSBundle mainBundle] preferredLocalizations].firstObject;
```
## Localized Currency Name
```objc
NSLocale *localizationLocale = [NSLocale localeWithLocaleIdentifier:currentLocalization];
NSString *yuanName = [localizationLocale displayNameForKey:NSLocaleCurrencyCode value: @“CNY”];
```
## Localized Quotations
```objc
NSLocale *localizationLocale = [NSLocale localeWithLocaleIdentifier:currentLocalization];
NSString *beginQuote = [localizationLocale displayNameForKey:NSLocaleQuotationBeginDelimiterKey];
NSString *endQuote = [localizationLocale displayNameForKey:NSLocaleQuotationEndDelimiterKey];
```
## Locale Change
Use `autoUpdatingCurrentLocale` instead of `currentLocale`. The standard locale-dependent APIs will handle updates.

For custom formatter templates, the listen for `NSCurrentLocaleDidChangeNotification` notifications. An example of responding to the notification:
```objc
[[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(userChangedLocale:) name:NSCurrentLocaleDidChangeNotification object:nil];

- (void)userChangedLocale:(NSNotification *)notification {
	formatter.locale = [NSLocale currentLocale]; // not using `autoUpdatingCurrentLocale`
	formatter.dateFormat = [NSDateFormatter dateFormatFromTemplate:template options:0 locale:formatter.locale];
	[view setNeedsDisplay];
}
```
## Calendar
Use `NSCalendar` for any calendrical calculations such as:
- number of days in a month
- components of a date
- delta computations

`NSDate` is a point in time and must be interpreted using an `NSCalendar`.

```objc
NSDateComponents *components = [[NSCalendar currentCalendar] components:NSDayCalendarUnit | NSMonthCalendarUnit fromDate:[NSDate date]];
NSInteger day = [components day];
NSInteger month = [components month];
```
## Images and Audio
Assets and other miscellaneous files can be localized by putting them in `.lproj` directories. The correct assets will be loaded when using standard APIs, including:
- `imageNamed:`
- `soundNamed:`
- `URLForResource:withExtension:`

## Unicode
Glyphs and characters have a many-to-many relationship. Operate on ranges, not characters, using standard APIs such as:
- `rangeOfComposedCharacterSequenceAtIndex:`
- `enumerateSubstringsInRange:options:usingBlock:` with options like `NSStringEnumerationByComposedCharacterSequences`
- `rangeOfString:options:range:locale:`
- `localizedStringCompare:`

## Sorting
Different languages have different sort orders and sometimes characters are considered as groups. Use `localizedStringCompare:` for any user strings.
## Text Input
Text is not always input letter by letter. In some languages, like Chinese, “marked” text is used and final character forms are picked from suggestions. Operate on text as it changes, instead of keystroke by keystroke. Inspect, but never change, `markedText`.
## Keyboard UI
Keyboards come in lots of different flavors. Never hardcode anything related to the keyboard. Use notifications to get information related to the current keyboard.
```objc
UIKeyboardWillShowNotification
UIKeyboardDidShowNotification
UIKeyboardWillHideNotification
UIKeyboardDidHideNotification
UIKeyboardWillChangeFrameNotification
UIKeyboardDidChangeFrameNotification
UIKeyboardAnimationCurveUserInfoKey
UIKeyboardAnimationDurationUserInfoKey
```
## Directionality
Use `NSTextAlignmentNatural` and `NSWritingDirectionNatural` whenever possible.
For complex cases, the translator should add left-to-right or right-to-left Unicode marks to localized strings.
## Sources
- http://nshipster.com/nslocalizedstring/
- http://www.objc.io/issues/9-strings/string-localization/
- https://developer.apple.com
