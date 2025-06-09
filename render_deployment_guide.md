# Deploying Dynamous Sample Python Agent to Render.com

This guide provides step-by-step instructions for deploying the Dynamous Sample Python Agent to Render.com using Supabase as the database backend.

## Prerequisites

- A Render.com account (free tier available)
- A Supabase account (free tier available)
- The repository forked/cloned to your GitHub account
- Basic understanding of environment variables and API endpoints

## Step 1: Setup Supabase Database

### 1.1 Create Supabase Project
1. Go to [supabase.com](https://supabase.com) and sign up/login
2. Click "New Project"
3. Choose your organization and provide:
   - **Project Name**: `dynamous-agent-db` (or your preferred name)
   - **Database Password**: Choose a strong password
   - **Region**: Select closest to your users
4. Click "Create new project" and wait for setup to complete

### 1.2 Get API Credentials
1. In your Supabase dashboard, navigate to **Settings** → **API**
2. Note down these values:
   - **Project URL**: `https://your-project-id.supabase.co`
   - **Service Role Key**: Long JWT token starting with `eyJ...`

### 1.3 Create Database Schema
1. In Supabase dashboard, go to **SQL Editor**
2. Click "New Query" and paste this SQL:

```sql
-- Enable the pgcrypto extension for UUID generation
CREATE EXTENSION IF NOT EXISTS pgcrypto;

-- Create messages table
CREATE TABLE messages (
    id uuid DEFAULT gen_random_uuid() PRIMARY KEY,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    session_id TEXT NOT NULL,
    message JSONB NOT NULL
);

-- Create indexes for better performance
CREATE INDEX idx_messages_session_id ON messages(session_id);
CREATE INDEX idx_messages_created_at ON messages(created_at);

-- Ensure service role has access (important for API access)
GRANT ALL ON messages TO service_role;
ALTER TABLE messages DISABLE ROW LEVEL SECURITY;
```

3. Click "Run" to execute the query

## Step 2: Prepare Repository Files

Ensure your repository contains these essential files:

### 2.1 requirements.txt
Create or verify this file contains:

```txt
fastapi>=0.104.0
uvicorn[standard]>=0.24.0
python-multipart>=0.0.6
pydantic>=2.0.0
supabase>=2.0.0
python-dotenv>=1.0.0
```

### 2.2 Optional: render.yaml
Create this file for automatic configuration:

```yaml
services:
  - type: web
    name: dynamous-python-agent
    env: python
    buildCommand: pip install -r requirements.txt
    startCommand: uvicorn sample_supabase_agent:app --host 0.0.0.0 --port $PORT
    plan: free
    envVars:
      - key: PORT
        value: 10000
```

## Step 3: Deploy to Render

### 3.1 Connect Repository
1. Log into your [Render dashboard](https://dashboard.render.com)
2. Click **"New"** → **"Web Service"**
3. Connect your GitHub account if not already connected
4. Select the `dynamous-sample-python-agent` repository

### 3.2 Configure Service Settings
Fill in the following configuration:

- **Name**: `dynamous-python-agent` (or your preferred name)
- **Region**: Choose closest to your users (e.g., Oregon, Frankfurt)
- **Branch**: `main` (or your preferred branch)
- **Runtime**: `Python 3`
- **Build Command**: `pip install -r requirements.txt`
- **Start Command**: `uvicorn sample_supabase_agent:app --host 0.0.0.0 --port $PORT`

### 3.3 Set Environment Variables
Click **"Advanced"** and add these environment variables:

```
SUPABASE_URL=https://your_url.supabase.co
SUPABASE_SERVICE_KEY=your_super_secret_key
API_BEARER_TOKEN=choose-a-secure-random-token
PORT=10000
```

| Key | Value | Description |
|-----|-------|-------------|
| `SUPABASE_URL` | `https://your-project-id.supabase.co` | Your Supabase project URL |
| `SUPABASE_SERVICE_KEY` | `eyJ...` | Your Supabase service role key |
| `API_BEARER_TOKEN` | `your-secure-random-token` | Choose a strong random token |
| `PORT` | `10000` | Port for the application |

**Important**: 
- Replace `your-project-id` with your actual Supabase project ID
- Replace `eyJ...` with your actual service role key
- Generate a strong random token for `API_BEARER_TOKEN`

### 3.4 Deploy
1. Click **"Create Web Service"**
2. Render will automatically:
   - Clone your repository
   - Install dependencies
   - Build and deploy your application
3. Wait for deployment to complete (usually 2-5 minutes)

## Step 4: Test Your Deployment

### 4.1 Get Your Application URL
Once deployed, Render provides a URL like: `https://your-app-name.onrender.com`

### 4.2 Test the API Endpoint
Use curl to test your deployed agent:

```bash
curl -X POST https://your-app-name.onrender.com/api/sample-supabase-agent \
-H "Authorization: Bearer your-secure-random-token" \
-H "Content-Type: application/json" \
-d '{
  "query": "Hello, agent!",
  "user_id": "test-user",
  "request_id": "test-request-1",
  "session_id": "test-session-1"
}'
```

**Expected Response:**
```json
{
  "success": true
}
```

### 4.3 Check API Documentation
Visit `https://your-app-name.onrender.com/docs` to see the automatic FastAPI documentation.

## Step 5: Monitoring and Maintenance

### 5.1 Monitor Logs
- Go to your Render dashboard
- Select your service
- Click **"Logs"** to view real-time application logs

### 5.2 Check Database
- In Supabase dashboard, go to **Table Editor** → **messages**
- Verify that test messages are being stored correctly

### 5.3 Performance Monitoring
- Monitor response times in Render dashboard
- Check Supabase usage in the Supabase dashboard

## Important Notes

### Free Tier Limitations
- **Render Free Tier**: Service spins down after 15 minutes of inactivity
- **Supabase Free Tier**: 500MB database, 2GB bandwidth/month
- First request after spin-down will be slower (cold start)

### Security Best Practices
- Never commit API keys or tokens to your repository
- Use strong, random tokens for `API_BEARER_TOKEN`
- Consider enabling Row Level Security (RLS) in Supabase for production
- Regularly rotate your API keys

### Scaling Considerations
- For production use, consider upgrading to paid plans
- Monitor database usage and optimize queries
- Set up proper error handling and logging

## Troubleshooting

### Common Issues

**Build Failures:**
- Check that all dependencies are listed in `requirements.txt`
- Verify Python version compatibility

**Database Connection Issues:**
- Verify Supabase URL format (no trailing slash)
- Ensure you're using the service role key, not anon key
- Check that the messages table exists

**Authentication Errors:**
- Verify `API_BEARER_TOKEN` matches between client and server
- Check Authorization header format: `Bearer your-token`

**404 Errors:**
- Ensure the endpoint path is correct: `/api/sample-supabase-agent`
- Check that the FastAPI app is properly configured

### Getting Help
- Check Render logs for detailed error messages
- Review Supabase logs in the dashboard
- Test Supabase connection independently using the REST API

## Example Production Configuration

For production deployments, consider:

```yaml
# render.yaml for production
services:
  - type: web
    name: dynamous-python-agent-prod
    env: python
    plan: starter  # Paid plan for better performance
    buildCommand: pip install -r requirements.txt
    startCommand: uvicorn sample_supabase_agent:app --host 0.0.0.0 --port $PORT --workers 2
    healthCheckPath: /health
    envVars:
      - key: PORT
        value: 10000
      - key: ENVIRONMENT
        value: production
```

## Conclusion

Your Dynamous Sample Python Agent is now deployed and accessible via the Render URL. The agent can handle requests, store conversation history in Supabase, and scale according to your needs.

For questions or issues, refer to:
- [Render Documentation](https://render.com/docs)
- [Supabase Documentation](https://supabase.com/docs)
- [FastAPI Documentation](https://fastapi.tiangolo.com/)