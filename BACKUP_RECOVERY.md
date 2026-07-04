# 💾 Backup & Recovery Guide

## Automated Daily Backups

The TRAVKA system automatically creates a backup every 24 hours.

Backups are stored in browser's localStorage with timestamp.

### Manual Backup Process

1. **Access Admin Panel:**
    - Click "🔐 Admin Panel" button
    - Enter password: `soprano67`

2. **Create Backup:**
    - Click "💾 Manual Backup" button
    - System confirms: "✅ Backup created successfully!"

3. **View Backups:**
    - Click "♻️ Restore Backup"
    - List shows all available backups with timestamps

### Recovery Procedure

⚠️ **Warning:** Recovery overwrites current data!

1. Navigate to admin panel
2. Click "♻️ Restore Backup"
3. Enter number of backup to restore
4. Confirm with "Yes" (second confirmation required)
5. System restores: "✅ Backup restored successfully!"

### Data Export

- **Format:** CSV (Excel compatible)
- **Click:** Admin → 📥 Export Data
- **File:** `travka_inquiries_YYYY-MM-DD.csv`
- **Contains:** Name, Email, Message, Timestamp

### Disaster Recovery

If localStorage is cleared by accident:

1. Browser cache is usually recoverable
2. Contact admin at @soprano67_1 on Telegram
3. Provide approx. time/date of incident
4. System can restore from GitHub backup

### Best Practices

✅ Do:
- Create manual backup weekly
- Export data to external storage monthly
- Review audit logs regularly
- Keep password secure
- Clear old backups periodically

❌ Don't:
- Share admin password
- Use same password elsewhere
- Delete backups without confirmation
- Export data to insecure locations

---

For issues: Contact @soprano67_1 on Telegram
