# Localization Fake Book
Fake book for localization on Apple platforms

## NSLocalizedString
Use these macros to take advantage of [genstrings](https://developer.apple.com/library/mac/documentation/Darwin/Reference/ManPages/man1/genstrings.1.html):

- `NSLocalizedString`
- `NSLocalizedStringFromTable`
- `NSLocalizedStringFromTableInBundle`
- `NSLocalizedStringWithDefaultValue`

All use NSBundle’s `localizedStringForKey:value:table:`

[genstrings](https://developer.apple.com/library/mac/documentation/Darwin/Reference/ManPages/man1/genstrings.1.html) usage:
```
find . -name *.m | xargs genstrings -o en.lproj
```
Use `-o` to specify output directory; “Localizable.strings” is default. Use `-a` to append.

For the localized string   
```  
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
```  
NSLocalizedString(“Run”, nil)  
```
However, this will result in a single entry in the strings file. Instead, do:
```
NSLocalizedString(“session.title.run”, nil)
NSLocalizedString(“precession.start.run”, nil)
```
so that the strings files have a key for each context.
## Format Strings 
To localize strings built at runtime:
```
[NSString localizedStringWithFormat:NSLocalizedString(@“session.component.reps.%lu out of reps %lu completed”, nil), completedReps, totalReps];
```
A translator can reorganize the format specifiers by referring to them using `%1$lu` and `%2$lu` in the strings file.
## Plural and Gender
Consider the example `@“%lu sessions”`. To appropriately handle plurals in English, there would need to be two different strings: `@“%lu sessions”` for when there are more than one session and `@“%lu session”` for when there is only one session. However, [other languages have different rules](http://www.unicode.org/cldr/charts/latest/supplemental/language_plural_rules.html).

To handle the complexity, we will use the typical `.strings` entry, normally generated like so:
```
[NSString localizedStringWithFormat:NSLocalizedString(@“profile.sessions-count.%lu sessions”, nil), completedReps, totalReps];
```
and a `.stringsdict.` `plist` file with the same name:
```
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
```
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
## Casing
To perform upper and lower casing, use `NSString`’s `lowercaseStringWithLocale:` and `uppercaseStringWithLocale:`.
## File Paths
Retrieve localized file paths using `NSURL`:
```
NSURL *url = [NSURL fileURLWithPath:@“/Applications/System Preferences.app”];
NSString *name;
[url getResourceValue:&name forKey:NSURLLocalizedTypeDescriptionKey error:NULL];
NSLog(@“localized name: %@“, name);
```
## Formatters
For numbers, use `NSNumberFormatter`.

For dates, use `NSDateFormatter`. For custom date and time styles, use `dateFormatFromTemplate:options:locale:` so that `NSDateFormatter` can figure out the correct format string based on the locale.  

Note that formatters are not thread safe. Using them from multiple threads is safe, but mutating them is not.

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
```
NSString *localization = [NSBundle mainBundle].preferredLocalizations.firstObject;
NSLocale *locale = [[NSLocale alloc] initWithLocaleIdentifier:localization];  
```

You can inspect locale instance properties, such as `NSLocaleUsesMetricSystem` using `objectForKey:`.
## Calendrical Calculations
Use `NSCalendar` for any calendrical calculations. One day is not equal to 86,400 seconds, for example.
## Images and Audio
Assets can be localized by putting them in `.lproj` directories. The correct assets will be loaded when using standard APIs, including `imageNamed:`, `soundNamed:`, and `URLForResource…`.
## Unicode
Glyphs and characters have a many-to-many relationship. Operate on ranges, not characters, using standard APIs such as:
- `rangeOfComposedCharacterSequence`
- `enumerateSubstringsInRange`
- `rangeOfString:options:range:locale:`
- `localizedStringCompare:`

## Sorting
Different languages have different sort orders and sometimes characters are considered as groups. Use `localizedStringCompare` for any user strings.
## Text Input
Text is not always input letter by letter. In some languages, like Chinese, “marked” text is used and final character forms are picked from suggestions. Operate on text as it changes, instead of keystroke by keystroke. 
## Sources
- http://nshipster.com/nslocalizedstring/
- http://www.objc.io/issues/9-strings/string-localization/
- https://developer.apple.com
