## Trade-Offs of Normalization

In the resume example, while the region_id field is a reference to a standard set of regions, the organization and school_name are just strings. This representation is denormalized: many people may have worked at the same company, but there is no ID linking them.

In a denormalized representation, we would include the image URL of the logo on every individual person's profile. This makes the JSON document self-contained, but it creates a headache if we ever need to change the logo, because we now need to find all the occurrences of the old URL and update them.

In the normalized representation, we would create an entity representing an organization or school and store its name, logo URL, and perhaps other attributes. Every resume that mentions the organization would then simply reference its ID, and updating the logo would be easy.

As a general principle, normalized data is usually faster to write (since there is only one copy) but slower to query (since it requires joins); denormalized data is usually faster to read (fewer joins) but more expensive to write (more copies to update, more disk space used).

Normalization tends to be better for OLTP systems, where both reads and updates need to be fast; analytical systems often fare better with denormalized data, since they perform updates in bulk and the performance of read-only queries is the dominant concern.

In systems of small to moderate scale, a normalized data model is often best because you don't have to worry about keeping multiple copies of the data consistent with one another, and the cost of performing joins is acceptable. However, in very large-scale systems, the cost of joins can become problematic.
