Notify Verifications and Comments Module
----------------------------------------

This optional module causes a notification to be created for each user when a record has
a comment added or a verification status change.

It depends on the Easy Login approach to identify recorders in order to associate
notifications with the correct users.

The notifications are added to the `notifications` table and can be optionally sent as an
email using the :doc:`notification-emails` module.