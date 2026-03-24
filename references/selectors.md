# X/Twitter DOM Selectors Reference

Last verified: March 2026

> **When X changes their markup**, update the selectors in this file.
> Everything else in the skill stays the same.

## Navigation & Layout

| Element | Selector | Notes |
|---------|----------|-------|
| Account switcher (sidebar) | `[data-testid="SideNav_AccountSwitcher_Button"]` | Contains username; use to verify login |
| Compose new tweet button | `[data-testid="SideNav_NewTweet_Button"]` | Opens compose dialog |
| Home tab | `[data-testid="AppTabBar_Home_Link"]` | Navigate to home feed |
| Search tab | `[data-testid="AppTabBar_Explore_Link"]` | |
| Notifications tab | `[data-testid="AppTabBar_Notifications_Link"]` | |
| Messages tab | `[data-testid="AppTabBar_DirectMessage_Link"]` | |

## Compose / Post

| Element | Selector | Notes |
|---------|----------|-------|
| Tweet text area | `[data-testid="tweetTextarea_0"]` | contenteditable div; focus then type |
| Post/Tweet button | `[data-testid="tweetButton"]` | In compose dialog |
| Close compose dialog | `[data-testid="app-bar-close"]` | X button on compose modal |
| Add media button | `[data-testid="fileInput"]` | Hidden file input for images/video |
| Emoji picker | `[data-testid="emojiButton"]` | |
| GIF picker | `[data-testid="gifSearchButton"]` | |
| Poll button | `[data-testid="pollButton"]` | |
| Schedule button | `[data-testid="scheduledButton"]` | |
| Location button | `[data-testid="geoButton"]` | |

## Tweet Actions (on individual tweets)

| Element | Selector | Notes |
|---------|----------|-------|
| Reply button | `[data-testid="reply"]` | |
| Retweet button | `[data-testid="retweet"]` | Opens dropdown |
| Retweet confirm | `[data-testid="retweetConfirm"]` | In dropdown after retweet click |
| Unretweet confirm | `[data-testid="unretweetConfirm"]` | |
| Like button | `[data-testid="like"]` | |
| Unlike button | `[data-testid="unlike"]` | Appears after liking |
| Share button | `[data-testid="share"]` | |
| Bookmark button | `[data-testid="bookmark"]` | |
| More options (⋯) | `[data-testid="caret"]` | Opens tweet options menu |

## Tweet Content (reading)

| Element | Selector | Notes |
|---------|----------|-------|
| Tweet container | `[data-testid="tweet"]` | Each tweet in a timeline |
| Tweet text | `[data-testid="tweetText"]` | The text content |
| User name display | `[data-testid="User-Name"]` | Display name + @handle |
| Tweet media | `[data-testid="tweetPhoto"]` | Images in tweet |
| Tweet video | `[data-testid="videoPlayer"]` | |

## Profile Page

| Element | Selector | Notes |
|---------|----------|-------|
| User name | `[data-testid="UserName"]` | |
| User description/bio | `[data-testid="UserDescription"]` | |
| Profile header items | `[data-testid="UserProfileHeader_Items"]` | Joined date, location, etc. |
| Edit profile button | `[data-testid="editProfileButton"]` | |
| Follow button | `[data-testid="followButton"]` | Follow/unfollow |

## Notifications & Toasts

| Element | Selector | Notes |
|---------|----------|-------|
| Toast notification | `[data-testid="toast"]` | Success/error messages |
| Toast text | `[data-testid="toastText"]` | The actual message |

## Search

| Element | Selector | Notes |
|---------|----------|-------|
| Search input | `[data-testid="SearchBox_Search_Input"]` | Main search bar |
| Search clear | `[data-testid="SearchBox_Search_Input_clear"]` | |

## Dialogs & Modals

| Element | Selector | Notes |
|---------|----------|-------|
| Confirmation dialog | `[data-testid="confirmationSheetDialog"]` | Generic confirm dialogs |
| Confirm button | `[data-testid="confirmationSheetConfirm"]` | |
| Cancel button | `[data-testid="confirmationSheetCancel"]` | |
| Mask/overlay | `[data-testid="mask"]` | Background overlay on modals |

---

## Fallback Strategy

If any selector returns `null`, use the `find` tool with natural language:

```
find("compose tweet button")
find("post button")  
find("reply button")
find("like button")
find("tweet text input")
```

The `find` tool uses accessibility tree matching, which is more resilient to
markup changes than CSS selectors. Use it as a fallback, then update this
file with the new selector once you discover what changed.
