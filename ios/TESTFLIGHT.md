# TestFlight — internal testing

**Current distribution path.** TellaVision is not on the App Store and is not
submitted for review. Builds go to a hand-picked internal group through
TestFlight. Public release is a later goal, not a cancelled one — see
`APP-STORE-LISTING.md`, which stays maintained for that day.

## Which kind of testing

Two paths, and the choice is about who the testers are, not how the build is made
— the same `.ipa` feeds both.

| | Internal | External |
|---|---|---|
| Who can test | Users on your App Store Connect team | Anyone with an email address or your public link |
| Do they need an account with you | **Yes** — they become team users | **No** — TestFlight app only |
| Apple review | None, ever | **Beta App Review**, per version |
| Turnaround | Minutes after processing | ~1-2 days for the first build of a version |
| Cap | 100 testers, 30 devices each | 10,000 testers |

**"Can I just send it to someone?"** — that is external testing. Internal is the
one that requires making people team users; external needs nothing from a tester
but the TestFlight app and an invite. Neither requires the tester to be an Apple
developer or to pay Apple anything.

External offers two ways in: **email invites**, where you enter addresses and
control exactly who, or a **public link**, which anyone can open. Both are
external; the link is not less private in terms of review, only in terms of who
can reach it.

The cost of external is one Beta App Review per version. Subsequent builds of an
already-reviewed version usually go out without waiting again, but Apple does not
guarantee it. This is also the first place a Guideline 4.2 "is this just a
website" conversation can happen — the answer is in `APP-STORE-LISTING.md`.

## One-time setup

1. **App record** — App Store Connect → My Apps → + → New App.
   Name `TellaVision`, English (U.S.), bundle ID `com.synergyav.tellavision`,
   SKU `TELLAVISION-001`. Leave every submission field blank; the record stays
   in "Prepare for Submission" indefinitely and TestFlight works regardless.

2. **Testers** — Users and Access → +. Internal testers must be users on the
   App Store Connect team; that is what "internal" means to Apple. Use the
   **Customer Support** role: the most limited one that still works for
   TestFlight, and it keeps testers out of certificates, banking and app
   metadata. They accept by email with an Apple ID.

3. **Group** — TestFlight → Internal Testing → new group → add testers → tick
   *automatically distribute new builds*, so later uploads need no clicks here.

## Every release

```bash
# build the web bundle first (build.html), then:
xcodebuild -project ios/TellaVision.xcodeproj -scheme TellaVision \
  -configuration Release -destination 'generic/platform=iOS' \
  -archivePath /tmp/tv-build/TellaVision.xcarchive archive -allowProvisioningUpdates

xcodebuild -exportArchive -archivePath /tmp/tv-build/TellaVision.xcarchive \
  -exportOptionsPlist ios/ExportOptions.plist \
  -exportPath /tmp/tv-build/export -allowProvisioningUpdates
```

Then upload `TellaVision.ipa` with Transporter. See `BUILD-NOTES.md` for the
signing traps.

**Bump `CURRENT_PROJECT_VERSION` every upload.** App Store Connect rejects a
duplicate build number for the same version outright, and the error arrives
after the upload rather than before it.

## Limits

- 100 internal testers, 30 devices each
- **Builds expire after 90 days** — re-upload quarterly or testers lose access
- `ITSAppUsesNonExemptEncryption = NO` is already in the Info.plist, so uploads
  skip the export-compliance prompt

## Setting up external testing

TestFlight → External Testing → new group → add testers by email, or enable a
public link. Then submit the build for Beta App Review from the same screen. You
will need a **beta app description**, a **feedback email**, and contact details
for the review — less than App Store submission asks for, and none of it public.
