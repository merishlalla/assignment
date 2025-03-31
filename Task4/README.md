## Simple hello world pipeline deployment

This goes through:
- Check code (ie does the code exist)
- Check if it's runable without errors
- If it exists then build using a dockerfile
- Then deploy. This isn't included in depth the file as a location to be deployed isn't specified. 

Assumptions: 
- Deployment would be triggered instead of an automatic deployment. (Example: If build fails, it shouldn't just go onto staging or production).
- 