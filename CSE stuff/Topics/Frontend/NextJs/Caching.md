even if you add setTimeout of a few seconds to a page. that still wont affect the actual page load time on production, because nextjs will pregenerate all the pages it can pregenerate.
hence, almost 0 load time for the pages.

downside, no refetch happens. things are cached. 

nexjts provides **revalidatePath()**
the path mentioned in this path will be revalidated. nested items wont be revalidated.

if do revalidatePath('/myPath', 'layout') -> then the layout will be revalidated, which in turn means all the nested pages will be revalidated as well.