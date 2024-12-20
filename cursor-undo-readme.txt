1 _About_
═════════

  Cursor-undo allows you to undo cursor movement commands using the
  Emacs standard `undo' command.

  For frequent cursor movements such as up/down/left/right, it combines
  the movements of the same direction into a single undo entry.  This
  prevents the undo command from reversing each individual character
  movement separately.  For example, if you move the cursor 20
  characters to the right and then 10 lines up, the first undo will go
  down 10 lines back, and the next undo move back 20 characters left.
  On the other hand, for search commands that often jump across multiple
  pages, each search command has its own undo entry, allowing you to
  undo them one at a time rather than as a combined operation.


2 _A "Brief" History_
═════════════════════

  This cursor-undo functionality has existed in my local Emacs init file
  for over 11+ years, since version 0 on 2013-06-26.  It was originally
  intended to support my ELPA package [BriefMode] only, but I later
  found it would be more useful if implemented in a more generalized
  way.  For years I have hoped for an official implementation of this
  feature, which is commonly seen among various editors.  Considering my
  implementation using advice functions a bit inelegant so I have always
  hesitated to release it till recently.

  Until there is official support for the cursor undo feature, this
  package serves most common daily needs.  The core design is to align
  with Emacs's native `undo' function by recording cursor positions and
  screen-relative position undo entries in the `buffer-undo-list' in
  accordance with its documentation.

  As this package primarily consists of advice functions to wrap cursor
  movement commands, each cursor movement command needs to be manually
  wrapped with `def-cursor-undo'.  For interactive functions that
  heavily invoke advised cursor movement commands, you may even need to
  advise them with `disable-cursor-tracking' to prevent generating
  numerous distinct cursor undo entries from a single command.  For user
  convenience, I have prepared ready `def-cursor-undo' advice sets for
  standard Emacs cursor movement commands, Brief Editor mode, Viper
  mode, and EVIL mode.


[BriefMode] <https://elpa.gnu.org/packages/brief.html>


3 _Usage_
═════════

  This package is released as a standard [ELPA package].  Just go to
  Emacs main menu -> "Options" -> "Manage Emacs Packages" and install
  it.  Once this package is installed, you only need to enable
  cursor-undo mode by adding the following line into your Emacs init
  file .emacs or init.el:

  ┌────
  │ (cursor-undo 1)
  └────

  That's all.  You can now move your cursor around and use stand Emacs
  `undo' (C-_, C-/ or C-x u) to move it back.  Even in read-only buffers
  you can still undo cursor movements.  This is convenient especially
  when browsing a huge file.


[ELPA package] <https://elpa.gnu.org/packages/cursor-undo.html>


4 _Notes for read-only buffers_
═══════════════════════════════

  Notice that the original `undo' cannot be performed in read-only
  buffers.  Here, I also advised the undo operation in read-only buffers
  as long as the pending undo list is still a cursor movement.

  This also enables a editing trick: If you are just editing a big file
  and moving the cursor to browse other parts but forgot where you were,
  you can undo cursor movements to go back to your last edited position
  by long-holding <undo> until the last editing command.  However, by
  doing so you risk missing your last edited operation as it might flash
  by so quickly that you don't even notice but keep undoing other cursor
  commands you don't want to undo.

  In this case, you can switch the buffer to read-only mode (by setting
  `buffer-read-only' to t), then long press <undo> until the undo
  command warns you that you're trying to edit a read-only buffer.  At
  this point, you're exactly at the latest editing position you are
  looking for.  You can then safely set `buffer-read-only' flag back to
  NIL and continue your editing.


5 _Notes for EVIL mode user_
════════════════════════════

  If you choose to use the default Emacs `undo' system, you should be
  able to use `evil-undo' to undo cursor movements.  If your choice is
  tree-undo or another undo system, you might need to use Emacs default
  `undo' (C-_, C-/ or C-x u …) to undo cursor movements.


6 _Notes for Viper mode user_
═════════════════════════════

  The default `viper-undo' is advised to allow cursor-undo.  If you find
  the advised function not working properly, consider comment out the
  following source code `(define-advice viper-undo …' to restore the
  original `viper-undo' function and use Emacs default `undo' (C-_, C-/
  or C-x u …)  to undo cursor movements.


7 _Technical notes_
═══════════════════

  Why do I use advice functions instead of the more generalized pre/post
  command hooks?  I bet you wouldn't like all the cursor movements in
  your debugging session being recorded as undo entries; when you start
  `undo'ing you will find the cursor replaying your previous debugging
  session in reverse order.  Similar effects happens in `semantic' mode
  and many others.  If it record *every* movement you will soon find it
  difficult to use in many situations.  Therefore, only the chosen
  editing key commands are advised with `def-cursor-undo'.  If you find
  something missing, just advise it with your own `def-cursor-undo'.


  Luke Lee
