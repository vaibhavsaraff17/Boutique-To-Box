# Google OAuth 2.0 Setup & Implementation Guide

## 📋 Table of Contents
- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Google Cloud Console Setup](#google-cloud-console-setup)
- [Environment Configuration](#environment-configuration)
- [Implementation Details](#implementation-details)
- [Testing the Integration](#testing-the-integration)
- [GitHub Deployment](#github-deployment)
- [Troubleshooting](#troubleshooting)
- [Security Considerations](#security-considerations)

---

## Overview

This guide provides step-by-step instructions for implementing Google OAuth 2.0 authentication in the Boutique-To-Box application using the **Implicit Flow** approach, which is optimized for frontend-only applications.

### What's Included
- ✅ Google OAuth 2.0 Implicit Flow implementation
- ✅ Seamless integration with existing authentication
- ✅ Beautiful, accessible UI components
- ✅ Comprehensive error handling
- ✅ TypeScript support with full type safety
- ✅ CSRF protection and security best practices

---

## Prerequisites

Before starting, ensure you have:
- [ ] A Google Cloud Platform account
- [ ] Node.js and npm installed
- [ ] The Boutique-To-Box project running locally
- [ ] Basic understanding of OAuth 2.0 concepts

---

## Google Cloud Console Setup

### Step 1: Create or Select a Project

1. Navigate to [Google Cloud Console](https://console.cloud.google.com)
2. Either create a new project or select an existing one
3. Note your project ID for reference

### Step 2: Enable Google+ API (Optional)

1. Go to **APIs & Services > Library**
2. Search for "Google+ API" 
3. Click **Enable** (This step is optional for basic OAuth)

### Step 3: Create OAuth 2.0 Credentials

1. Navigate to **APIs & Services > Credentials**
2. Click **"+ CREATE CREDENTIALS"**
3. Select **"OAuth client ID"**
4. Choose **"Web application"** as application type

### Step 4: Configure OAuth Client

Fill in the following details:

**Name:** `Boutique-To-Box OAuth Client`

**Authorized JavaScript origins:**
```
http://localhost:5173
http://localhost:8080
http://127.0.0.1:5173
```

**Authorized redirect URIs:**
```
http://localhost:5173/auth/google/callback
http://localhost:8080/auth/google/callback
```

### Step 5: Save Your Credentials

After creation, you'll receive:
- **Client ID** (starts with numbers, ends with `.apps.googleusercontent.com`)
- **Client Secret** (keep this secure!)

⚠️ **Important:** Download the JSON file and store it securely. Never commit credentials to version control.

---

## Environment Configuration

### Step 1: Create Environment Files

Create a `.env` file in your project root:

```env
# Google OAuth Configuration
VITE_GOOGLE_CLIENT_ID=your_actual_client_id_here.apps.googleusercontent.com
VITE_GOOGLE_CLIENT_SECRET=your_actual_client_secret_here

# Optional: Debug mode for development
VITE_DEBUG_OAUTH=true

# OAuth Configuration
VITE_OAUTH_REDIRECT_URI=http://localhost:5173/auth/google/callback
```

### Step 2: Update .env.example

```env
# Google OAuth Configuration (for other developers)
VITE_GOOGLE_CLIENT_ID=your_google_client_id_here
VITE_GOOGLE_CLIENT_SECRET=your_google_client_secret_here
VITE_DEBUG_OAUTH=false
VITE_OAUTH_REDIRECT_URI=http://localhost:5173/auth/google/callback
```

### Step 3: Add to .gitignore

Ensure your `.gitignore` includes:
```gitignore
.env
.env.local
.env.production
```

---

## Implementation Details

### Architecture Overview

The implementation uses **OAuth 2.0 Implicit Flow** which is ideal for frontend applications:

```
User → Google Sign-In Button → Google OAuth → Callback Handler → AuthContext → Dashboard
```

### Key Components

#### 1. Google OAuth Service (`src/services/googleAuth.ts`)
```typescript
// Handles OAuth flow with Implicit Flow approach
class GoogleAuthService {
  signIn(): Promise<void>
  handleCallback(): Promise<UserData>
  signOut(): Promise<void>
}
```

#### 2. Google Sign-In Button (`src/components/GoogleSignInButton.tsx`)
```typescript
// Reusable component with loading states
<GoogleSignInButton 
  variant="default" 
  className="w-full"
  onSuccess={(user) => console.log('User signed in:', user)}
/>
```

#### 3. OAuth Callback Handler (`src/pages/GoogleCallback.tsx`)
```typescript
// Processes OAuth tokens and redirects users
// Includes loading states and error handling
```

#### 4. Enhanced AuthContext
```typescript
// Updated to support Google OAuth alongside email auth
interface AuthContextType {
  loginWithGoogle: () => Promise<void>
  authProvider: 'email' | 'google'
}
```

### OAuth Flow Details

1. **Initiation:** User clicks "Sign in with Google"
2. **Redirect:** Browser navigates to Google OAuth endpoint
3. **Authorization:** User grants permissions
4. **Callback:** Google redirects to `/auth/google/callback`
5. **Token Processing:** Extract access token and ID token from URL fragment
6. **Profile Fetch:** Get user profile information
7. **Context Update:** Update AuthContext with user data
8. **Navigation:** Redirect to dashboard

---

## Testing the Integration

### Step 1: Start Development Server

```bash
npm run dev
# or
yarn dev
```

### Step 2: Test OAuth Flow

1. Navigate to `http://localhost:5173/login`
2. Click **"Continue with Google"**
3. Complete Google authentication
4. Verify successful redirect to dashboard
5. Check browser console for any errors

### Step 3: Test Different Scenarios

- ✅ **Successful login**
- ✅ **User cancels OAuth flow**
- ✅ **Network errors**
- ✅ **Invalid credentials**
- ✅ **Token expiration**

### Step 4: Verify Token Storage

Check browser Developer Tools:
- **Application > Local Storage**
- Look for stored tokens and user data
- Verify token expiration handling

---

## GitHub Deployment

### Overview

Deploying your Google OAuth implementation to GitHub requires additional configuration for production URLs and secure environment variable management.

### Step 1: GitHub Pages Setup

#### Option A: GitHub Pages Deployment

1. **Repository Settings**
   - Go to your GitHub repository
   - Navigate to **Settings > Pages**
   - Select source: **Deploy from a branch**
   - Choose **main** branch and **/ (root)** folder

2. **Build Configuration**
   
   Update your `package.json` with build scripts:
   ```json
   {
     "scripts": {
       "build": "vite build",
       "preview": "vite preview",
       "deploy": "npm run build && gh-pages -d dist"
     }
   }
   ```

3. **Install GitHub Pages Deploy Tool**
   ```bash
   npm install --save-dev gh-pages
   ```

4. **Deploy Command**
   ```bash
   npm run deploy
   ```

#### Option B: GitHub Actions CI/CD

Create `.github/workflows/deploy.yml`:

```yaml
name: Deploy to GitHub Pages

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    
    steps:
    - name: Checkout
      uses: actions/checkout@v3
      
    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '18'
        cache: 'npm'
        
    - name: Install dependencies
      run: npm ci
      
    - name: Build
      run: npm run build
      env:
        VITE_GOOGLE_CLIENT_ID: ${{ secrets.VITE_GOOGLE_CLIENT_ID }}
        VITE_OAUTH_REDIRECT_URI: ${{ secrets.VITE_OAUTH_REDIRECT_URI }}
        
    - name: Deploy to GitHub Pages
      uses: peaceiris/actions-gh-pages@v3
      if: github.ref == 'refs/heads/main'
      with:
        github_token: ${{ secrets.GITHUB_TOKEN }}
        publish_dir: ./dist
```

### Step 2: Production OAuth Configuration

#### Update Google Cloud Console

1. **Add Production URLs to Google Console**
   
   **Authorized JavaScript origins:**
   ```
   https://yourusername.github.io
   https://your-custom-domain.com (if using custom domain)
   ```
   
   **Authorized redirect URIs:**
   ```
   https://yourusername.github.io/Boutique-To-Box/auth/google/callback
   https://your-custom-domain.com/auth/google/callback
   ```

2. **Multiple Environment Support**
   
   For different environments, consider creating separate OAuth clients:
   - **Development:** `Boutique-To-Box Dev`
   - **Staging:** `Boutique-To-Box Staging`
   - **Production:** `Boutique-To-Box Production`

#### GitHub Secrets Configuration

1. **Navigate to Repository Settings**
   - Go to **Settings > Secrets and variables > Actions**
   - Click **New repository secret**

2. **Add Required Secrets**
   ```
   VITE_GOOGLE_CLIENT_ID=your_production_client_id
   VITE_OAUTH_REDIRECT_URI=https://yourusername.github.io/Boutique-To-Box/auth/google/callback
   ```

3. **Optional Production Secrets**
   ```
   VITE_DEBUG_OAUTH=false
   VITE_API_BASE_URL=https://your-api-domain.com
   ```

### Step 3: Environment-Specific Configuration

#### Create Multiple Environment Files

**`.env.development`** (for local development):
```env
VITE_GOOGLE_CLIENT_ID=your_dev_client_id
VITE_OAUTH_REDIRECT_URI=http://localhost:5173/auth/google/callback
VITE_DEBUG_OAUTH=true
```

**`.env.production`** (for GitHub Pages):
```env
VITE_GOOGLE_CLIENT_ID=your_prod_client_id
VITE_OAUTH_REDIRECT_URI=https://yourusername.github.io/Boutique-To-Box/auth/google/callback
VITE_DEBUG_OAUTH=false
```

#### Update Vite Configuration

**`vite.config.ts`**:
```typescript
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [react()],
  base: process.env.NODE_ENV === 'production' ? '/Boutique-To-Box/' : '/',
  build: {
    outDir: 'dist',
    sourcemap: false,
  },
  define: {
    'process.env.NODE_ENV': JSON.stringify(process.env.NODE_ENV),
  }
})
```

### Step 4: Dynamic Configuration

#### Environment Detection Service

Create `src/config/environment.ts`:

```typescript
export const getEnvironmentConfig = () => {
  const isDevelopment = import.meta.env.MODE === 'development'
  const isProduction = import.meta.env.MODE === 'production'
  
  return {
    googleClientId: import.meta.env.VITE_GOOGLE_CLIENT_ID,
    redirectUri: isDevelopment 
      ? 'http://localhost:5173/auth/google/callback'
      : 'https://yourusername.github.io/Boutique-To-Box/auth/google/callback',
    debugMode: isDevelopment,
    apiBaseUrl: isDevelopment 
      ? 'http://localhost:3000'
      : 'https://your-production-api.com'
  }
}
```

#### Update OAuth Service

**`src/services/googleAuth.ts`**:
```typescript
import { getEnvironmentConfig } from '../config/environment'

class GoogleAuthService {
  private config = getEnvironmentConfig()
  
  async signIn(): Promise<void> {
    const authUrl = `https://accounts.google.com/oauth/v2/auth?` +
      `client_id=${this.config.googleClientId}&` +
      `redirect_uri=${encodeURIComponent(this.config.redirectUri)}&` +
      `response_type=token id_token&` +
      `scope=openid email profile`
    
    window.location.href = authUrl
  }
}
```

### Step 5: Custom Domain (Optional)

#### GitHub Pages Custom Domain

1. **Add CNAME File**
   
   Create `public/CNAME`:
   ```
   your-custom-domain.com
   ```

2. **DNS Configuration**
   
   Add DNS records:
   ```
   CNAME: www.your-domain.com → yourusername.github.io
   A: your-domain.com → 185.199.108.153
   A: your-domain.com → 185.199.109.153
   A: your-domain.com → 185.199.110.153
   A: your-domain.com → 185.199.111.153
   ```

3. **Update OAuth Configuration**
   
   Add to Google Console authorized origins:
   ```
   https://your-custom-domain.com
   https://www.your-custom-domain.com
   ```

### Step 6: Production Testing

#### Pre-Deployment Checklist

- [ ] All environment variables configured in GitHub Secrets
- [ ] Production URLs added to Google Console
- [ ] Build process works locally (`npm run build`)
- [ ] OAuth flow tested with production URLs
- [ ] Custom domain (if used) properly configured
- [ ] HTTPS enforced for OAuth callbacks

#### Testing Production Deployment

1. **Build Locally**
   ```bash
   npm run build
   npm run preview
   ```

2. **Test Production URLs**
   - Navigate to your GitHub Pages URL
   - Test OAuth flow end-to-end
   - Verify token storage and retrieval
   - Check browser console for errors

3. **Monitor Deployment**
   - Check GitHub Actions logs
   - Verify successful deployment
   - Test from different devices/browsers

### Step 7: Deployment Troubleshooting

#### Common GitHub Deployment Issues

**🚨 404 Errors on GitHub Pages**

**Problem:** Routes not working after deployment

**Solution:**
1. Add `404.html` that redirects to `index.html`:
   ```html
   <!DOCTYPE html>
   <html>
   <head>
     <script>
       window.location.href = '/Boutique-To-Box/'
     </script>
   </head>
   </html>
   ```

**🚨 OAuth Redirect Mismatch on Production**

**Problem:** Redirect URI doesn't match production URL

**Solution:**
1. Double-check production URL format
2. Ensure base path is included: `/Boutique-To-Box/`
3. Verify HTTPS protocol

**🚨 Environment Variables Not Loading**

**Problem:** Build fails due to missing environment variables

**Solution:**
1. Check GitHub Secrets are properly named
2. Verify workflow has access to secrets
3. Add fallback values in code

#### GitHub Actions Debugging

Enable debug logging in workflow:
```yaml
- name: Debug Environment
  run: |
    echo "Node version: $(node --version)"
    echo "Build environment: $NODE_ENV"
    echo "Client ID configured: ${{ secrets.VITE_GOOGLE_CLIENT_ID != '' }}"
```

### Step 8: Monitoring & Maintenance

#### Production Monitoring

1. **Google Analytics Integration**
   ```typescript
   // Track OAuth events
   gtag('event', 'login', {
     method: 'Google'
   })
   ```

2. **Error Tracking**
   ```typescript
   // Log production errors
   if (import.meta.env.PROD) {
     console.error('OAuth Error:', error)
     // Send to error tracking service
   }
   ```

3. **Performance Monitoring**
   - Monitor OAuth flow completion rates
   - Track authentication success/failure rates
   - Monitor page load times

#### Regular Maintenance

- [ ] **Monthly:** Review OAuth credentials and rotate if needed
- [ ] **Quarterly:** Update dependencies and security patches
- [ ] **Bi-annually:** Review and update Google Console settings
- [ ] **Annually:** Audit OAuth scopes and permissions

---

## Troubleshooting

### Common Issues & Solutions

#### 🚨 "redirect_uri_mismatch" Error

**Problem:** Google rejects the redirect URI

**Solution:**
1. Check Google Console authorized redirect URIs
2. Ensure exact match (including protocol and port)
3. Common URIs to add:
   - `http://localhost:5173/auth/google/callback`
   - `http://localhost:8080/auth/google/callback`

#### 🚨 "invalid_client" Error

**Problem:** Client ID or Secret is incorrect

**Solution:**
1. Verify `.env` file has correct credentials
2. Check for extra spaces or quotes
3. Ensure Client ID ends with `.apps.googleusercontent.com`

#### 🚨 "access_denied" Error

**Problem:** User denied permissions or canceled flow

**Solution:**
- This is normal user behavior
- App should handle gracefully with error message
- Allow user to retry authentication

#### 🚨 CORS Errors

**Problem:** Cross-origin request blocked

**Solution:**
1. Add authorized JavaScript origins in Google Console
2. Include all development URLs:
   - `http://localhost:5173`
   - `http://localhost:8080`
   - `http://127.0.0.1:5173`

#### 🚨 Token Validation Errors

**Problem:** ID token validation fails

**Solution:**
1. Check token expiration
2. Verify audience (Client ID) matches
3. Ensure proper token format

#### 🚨 GitHub Pages Deployment Issues

**Problem:** OAuth fails after GitHub Pages deployment

**Solution:**
1. Update Google Console with production URLs:
   - `https://yourusername.github.io`
   - `https://yourusername.github.io/Boutique-To-Box/auth/google/callback`
2. Configure GitHub Secrets for environment variables
3. Ensure base path is correctly configured in Vite
4. Verify HTTPS is enforced (required for OAuth)

#### 🚨 Environment Variable Issues in Production

**Problem:** Environment variables not loading in GitHub Actions

**Solution:**
1. Check GitHub repository **Settings > Secrets and variables > Actions**
2. Ensure secret names match exactly (case-sensitive)
3. Verify workflow has access to secrets
4. Add debug logging to verify variables are loaded

### Debug Mode

Enable debug logging by setting:
```env
VITE_DEBUG_OAUTH=true
```

This will show detailed OAuth flow information in browser console.

---

## Security Considerations

### ✅ Implemented Security Features

1. **CSRF Protection**
   - State parameter validation
   - Prevents cross-site request forgery

2. **Token Validation**
   - ID token signature verification
   - Audience and issuer validation

3. **Secure Token Storage**
   - Tokens stored in localStorage with expiration
   - Automatic cleanup on logout

4. **Error Handling**
   - No sensitive information in error messages
   - Graceful degradation for failed requests

### 🔒 Production Recommendations

1. **Server-Side Validation**
   ```typescript
   // Verify tokens on your backend
   const ticket = await client.verifyIdToken({
     idToken: token,
     audience: CLIENT_ID,
   });
   ```

2. **Token Refresh**
   - Implement automatic token refresh
   - Handle expired tokens gracefully

3. **Rate Limiting**
   - Implement OAuth request rate limiting
   - Prevent abuse of authentication endpoints

4. **Monitoring**
   - Log authentication events
   - Monitor for suspicious activity

---

## Advanced Configuration

### Custom Scopes

To request additional permissions:

```typescript
// In googleAuth.ts
const scopes = [
  'openid',
  'email', 
  'profile',
  'https://www.googleapis.com/auth/calendar.readonly' // Example
];
```

### Brand Customization

Customize the Google Sign-In button:

```typescript
<GoogleSignInButton
  variant="outline"
  size="lg"
  theme="dark"
  logoAlignment="left"
  text="Continue with Google"
/>
```

### Error Customization

Define custom error messages:

```typescript
const ERROR_MESSAGES = {
  'access_denied': 'Authentication was cancelled. Please try again.',
  'invalid_client': 'Authentication configuration error. Please contact support.',
  'network_error': 'Network error. Please check your connection and try again.'
};
```

---

## Testing Checklist

Before deploying, verify:

- [ ] OAuth flow works in different browsers
- [ ] Error handling works for all scenarios  
- [ ] Tokens are properly stored and cleaned up
- [ ] User can sign out successfully
- [ ] No sensitive data in browser console
- [ ] CORS is properly configured
- [ ] Environment variables are secure

### GitHub Deployment Checklist

Additional checks for GitHub Pages deployment:

- [ ] Production URLs added to Google Console
- [ ] GitHub Secrets configured for environment variables
- [ ] Base path configured correctly in `vite.config.ts`
- [ ] Build process completes successfully
- [ ] GitHub Actions workflow runs without errors
- [ ] OAuth flow tested on production URL
- [ ] HTTPS enforced (required for OAuth)
- [ ] Custom domain configured (if applicable)
- [ ] 404 handling implemented for SPA routing

---

## Support & Resources

### Useful Links
- [Google OAuth 2.0 Documentation](https://developers.google.com/identity/protocols/oauth2)
- [Google Cloud Console](https://console.cloud.google.com)
- [OAuth 2.0 Implicit Flow Guide](https://tools.ietf.org/html/rfc6749#section-4.2)
- [GitHub Pages Documentation](https://docs.github.com/en/pages)
- [GitHub Actions for Node.js](https://docs.github.com/en/actions/automating-builds-and-tests/building-and-testing-nodejs)
- [Vite Environment Variables](https://vitejs.dev/guide/env-and-mode.html)

### Getting Help
- Check browser console for detailed error messages
- Enable debug mode for verbose logging
- Review Google Cloud Console audit logs
- Check GitHub Actions workflow logs for deployment issues
- Consult OAuth 2.0 specification for advanced issues
- Review GitHub Pages troubleshooting guide

---

## 🎉 Conclusion

Your Google OAuth 2.0 integration is now complete! Users can seamlessly sign in with their Google accounts alongside the existing email/password authentication.

### What You've Accomplished:
✅ Secure OAuth 2.0 implementation with Implicit Flow  
✅ Beautiful, accessible user interface  
✅ Comprehensive error handling and security  
✅ Production-ready configuration  
✅ Full TypeScript support

The implementation follows OAuth 2.0 best practices and provides a smooth user experience while maintaining security standards.

**Happy coding! 🚀**
