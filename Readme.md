# Install Redis server and client in Google Colab
!apt-get install redis-server
!redis-server --daemonize yes
!pip install redis



import json  # Importing the JSON module for handling JSON data
import redis  # Importing the Redis library for interacting with Redis

class BlogContentCache:
    def __init__(self, redis_host='localhost', redis_port=6379, redis_db=0):
        try:
            self.redis_client = redis.StrictRedis(host=redis_host, port=redis_port, db=redis_db)
            self.redis_client.ping()  # Test the connection
            print("Connected to Redis successfully.")
        except redis.ConnectionError as e:
            print(f"Failed to connect to Redis: {e}")
            self.redis_client = None

    def get_post(self, post_id):
        if not self.redis_client:
            print("Redis client is not available.")
            return None

        # Check if the post is in the cache
        cached_post = self.redis_client.get(f'post:{post_id}')

        if cached_post:
            # If the post is cached, retrieve it from cache
            print("Retrieving post from cache.")
            return json.loads(cached_post.decode('utf-8'))
        else:
            # If the post is not cached, fetch it from the database (simulated)
            post = {'post_id': post_id, 'title': f'Title of Post {post_id}', 'content': f'Content of Post {post_id}'}
            
            # Store the post in the cache for future retrieval
            self.redis_client.set(f'post:{post_id}', json.dumps(post))
            print("Fetching post from the database and caching.")
            return post

# Example Usage
if __name__ == "__main__":
    # Create an instance of the BlogContentCache
    blog_cache = BlogContentCache()

    # Fetch and cache blog posts
    post_id_1 = 1
    post_id_2 = 2

    # Fetch the first post
    post_1 = blog_cache.get_post(post_id_1)
    print(f"Post: {post_1}")

    # Fetch the first post again to demonstrate caching
    cached_post_1 = blog_cache.get_post(post_id_1)
    print(f"Cached Post: {cached_post_1}")

    # Fetch a different post
    post_2 = blog_cache.get_post(post_id_2)
    print(f"Post: {post_2}")

    # Fetch the second post again to demonstrate caching
    cached_post_2 = blog_cache.get_post(post_id_2)
    print(f"Cached Post: {cached_post_2}")
