# App: Context, Variables, Dialogs, File Panels

## Context management
```bash
cli-anything-iterm2 --json app status                  # inventory all windows/tabs/sessions
cli-anything-iterm2 app current                        # focus → saves window/tab/session as context
cli-anything-iterm2 app context                        # show saved context
cli-anything-iterm2 app set-context --session-id <id>
cli-anything-iterm2 app clear-context
```

## App-level variables
```bash
cli-anything-iterm2 app get-var hostname
cli-anything-iterm2 app set-var user.myvar hello
```

## Modal dialogs
```bash
cli-anything-iterm2 app alert "Title" "Message"
cli-anything-iterm2 app alert "Deploy?" "Push?" --button Yes --button No
cli-anything-iterm2 app text-input "Rename" "Enter name:" --default "myapp"
```

## File panels
```bash
cli-anything-iterm2 app file-panel                              # macOS open picker
cli-anything-iterm2 app file-panel --ext py --ext txt --multi   # filter + multi-select
cli-anything-iterm2 app save-panel --filename output.txt        # save dialog
```
