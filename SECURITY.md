# Security Guide

## Security Improvements Implemented

### 1. Authentication & Authorization
- **HTTP Basic Authentication** on all admin endpoints
- Default credentials: admin/SecurePassword123!
- Environment variables: `ADMIN_USERNAME` and `ADMIN_PASSWORD`
- Protected endpoints: `/dashboard`, `/candidates`, `/settings/*`, `/export/*`, `/retry/*`

### 2. File Upload Security
- **File size limit**: 10MB maximum
- **MIME type validation**: Uses python-magic when available, falls back to file signatures
- **Extension validation**: Ensures extension matches content
- **Secure filename handling**: Uses werkzeug.secure_filename()
- **Allowed types only**: PDF, DOC, DOCX

### 3. Server Configuration
- **Debug mode disabled** by default (use `FLASK_DEBUG=true` to enable)
- **Host binding**: Defaults to 127.0.0.1 (localhost only)
- **Environment-based configuration**

### 4. Error Handling
- **Specific exception handling**: Replaced bare `except:` clauses
- **No sensitive data in error messages**
- **Proper logging without exposing secrets**

## Environment Variables

Create/update your `.env` file:

```bash
# Required
OPENAI_API_KEY=your_openai_api_key_here
ZOHO_FLOW_WEBHOOK=your_zoho_webhook_url_here

# Security (recommended to change defaults)
ADMIN_USERNAME=admin
ADMIN_PASSWORD=YourSecurePasswordHere123!

# Server Configuration
FLASK_DEBUG=false
FLASK_HOST=127.0.0.1
```

## Production Deployment Checklist

### Pre-deployment
- [ ] Change default admin password
- [ ] Set `FLASK_DEBUG=false`
- [ ] Review and secure environment variables
- [ ] Ensure `.env` file is not committed to version control
- [ ] Install system dependencies (libmagic for advanced file validation)

### Production Server Setup
```bash
# Install system dependencies
brew install libmagic  # macOS
# or
sudo apt-get install libmagic1  # Ubuntu/Debian

# Install Python dependencies
pip install -r requirements.txt

# Set production environment variables
export FLASK_DEBUG=false
export FLASK_HOST=0.0.0.0  # Only if you need external access
export ADMIN_PASSWORD=YourSecurePasswordHere123!

# Use a production WSGI server
pip install gunicorn
gunicorn -w 4 -b 0.0.0.0:5001 app:app
```

### Additional Security Recommendations

1. **Use HTTPS in production**
   - Configure SSL certificates
   - Force HTTPS redirects

2. **Implement proper session management**
   - Consider JWT tokens instead of Basic Auth
   - Add session timeouts

3. **Add rate limiting**
   - Prevent brute force attacks
   - Limit file upload frequency

4. **Database security**
   - Move from JSON file to proper database
   - Encrypt sensitive data at rest

5. **Monitoring & Logging**
   - Set up proper application logging
   - Monitor failed authentication attempts
   - Alert on suspicious activities

## Security Testing

### Authentication Test
```bash
# Should return 401 Unauthorized
curl -i http://localhost:5001/dashboard

# Should return 200 OK
curl -i -u admin:YourPassword http://localhost:5001/dashboard
```

### File Upload Test
```bash
# Test file size limit (create 11MB file)
dd if=/dev/zero of=large_file.pdf bs=1M count=11
curl -X POST -F "file=@large_file.pdf" http://localhost:5001/upload
# Should return 400 Bad Request with file size error
```

## Incident Response

If you suspect a security breach:
1. Immediately change admin credentials
2. Review server logs for unauthorized access
3. Check uploaded files for malicious content
4. Rotate API keys (OpenAI, Zoho)
5. Update all dependencies

## Contact

For security issues, please review the application logs and update credentials immediately.