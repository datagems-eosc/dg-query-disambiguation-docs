# FAQ & Known Issues

Frequently asked questions and known issues for the Query Disambiguation Service.

## General Questions

### What is the Query Disambiguation Service?

The Query Disambiguation Service analyzes natural language queries to identify ambiguities, provide rephrased options, and suggest complete clarified queries. It's particularly focused on spatial and temporal information.

### What models does it support?

The service uses OpenAI models. The default is `gpt-4o-mini`, but you can configure other models via the `OPENAI_MODEL_NAME` environment variable.

### Is authentication required?

Currently, the service does not require authentication. All endpoints are publicly accessible. See [Security](./security.md) for details.

## Usage Questions

### How do I resolve relative time references?

The service automatically resolves relative time references (e.g., "last month", "yesterday") based on the current date/time. You can optionally provide a `current_datetime` parameter in the request, or the service will use the system's current time.

### What is the maximum query length?

The maximum query length is 10,000 characters.

### How long does disambiguation take?

Response time depends on:
- Query complexity
- OpenAI API latency
- Model used

Typical response time is 2-5 seconds.

### Can I use my own LLM?

The service is currently designed for OpenAI models. To use other providers, you would need to modify the `QueryDisambiguationTool` class in `src/tools/query_disambiguation.py`.

## Technical Questions

### How do I change the OpenAI model?

Set the `OPENAI_MODEL_NAME` environment variable:
```bash
export OPENAI_MODEL_NAME='gpt-4o'
```

Or pass it when initializing the tool:
```python
tool = QueryDisambiguationTool(model_name='gpt-4o')
```

### How do I handle errors?

The service returns standard HTTP status codes:
- `200`: Success
- `400`: Bad request
- `422`: Validation error
- `500`: Server error

See [Error Codes](./error-codes.md) for details.

### Can I run multiple instances?

Yes, the service is stateless and can be horizontally scaled. Run multiple instances behind a load balancer.

## Known Issues

### Issue: Slow response times

**Cause**: OpenAI API latency or rate limiting

**Solution**:
- Check OpenAI API status
- Verify API key has sufficient quota
- Consider using a faster model (e.g., gpt-4o-mini)
- Implement caching for frequent queries

### Issue: Inconsistent results

**Cause**: LLM non-determinism

**Solution**:
- Lower the temperature parameter (default: 0.3)
- Use a more deterministic model
- Implement result caching

### Issue: High API costs

**Cause**: Frequent API calls to OpenAI

**Solution**:
- Use gpt-4o-mini (more cost-effective)
- Implement caching
- Monitor usage and set budgets
- Consider batch processing

### Issue: Timezone handling

**Current behavior**: Uses system timezone

**Workaround**: Provide explicit `current_datetime` in requests with timezone information

## Troubleshooting

### Service won't start

1. Check `OPENAI_API_KEY` is set
2. Verify Python version (3.11+)
3. Check port 8000 is available
4. Review error logs

### API returns 500 errors

1. Check OpenAI API key is valid
2. Verify API key has quota
3. Check service logs for details
4. Verify network connectivity to OpenAI

### Results are inaccurate

1. Try a different model
2. Adjust temperature parameter
3. Provide more context in queries
4. Check if query is too ambiguous

## Performance

### How many requests per second can it handle?

Depends on:
- OpenAI API rate limits
- Server resources
- Query complexity

For production, implement rate limiting and consider horizontal scaling.

### How do I improve performance?

1. Use faster models (gpt-4o-mini)
2. Implement caching
3. Scale horizontally
4. Optimize queries

## Cost Management

### How much does it cost?

Costs depend on:
- Model used (gpt-4o-mini is cheaper)
- Query length
- Number of requests

Monitor usage via OpenAI dashboard.

### How do I reduce costs?

1. Use gpt-4o-mini
2. Implement caching
3. Batch requests when possible
4. Set usage limits

## Support

### Where can I get help?

- Check documentation
- Review GitHub issues
- Contact maintainers
- Check OpenAI API documentation

### How do I report a bug?

Create an issue on GitHub with:
- Description of the issue
- Steps to reproduce
- Expected vs actual behavior
- Environment details
- Relevant logs
